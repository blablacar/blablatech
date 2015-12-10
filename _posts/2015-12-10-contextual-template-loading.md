---
layout:         post
title:          Contextual template loading
tags:           [application-architecture, technology]
authors:        [pierre-yves-lebecq]
description:    The full story of the second Coding Night at BlaBlaCar.
---

# Goal

On the BlaBlaCar platform, we have some pages and emails which are very contextual. It depends on the booking mode for the trip (online vs. onboard), and sometimes the payment mode (online, online no fees, onboard, onboard no payment and also sometimes credit card vs. paypal, etc.). And also, if you are on web or web mobile you will want a different rendering.
The context is used to render web or mobile web templates, display or not some parts of pages, use different translation keys, etc.

Loading of mobile web templates was done in the `Blablacar\Bundle\MainBundle\Controller\Controller::render()` method. It means that every time you asked twig to render a template outside a controller, or you used twig `render()` or `include()` functions, you had to know if you wanted to refer to a web mobile or web template by yourself.

Altering the rendering on a template was done using many twig `if` statements, combined with `include()` or `render()` calls to make the render different for the context we have.
This made some templates overly cluttered, difficult to read, difficult to maintain.

For the i18n part, we used a custom translator handling a system of key suffixes. You can set suffixes on the translator, and when you want to get the translation for a given key, it will add the suffixes to the key and try to find a translation. If it can't find a corresponding translation, it will remove the last set suffix and tries again to find a corresponding translation. And it will proceed this way until falling back to the original key you asked. This makes very hard to know which translations keys we have in database are really in use and which are not, and also where are they used. You can't just grep for a given translation key to find it, because they are likely to be dynamically generated using suffixes.

# Where did we want to go?

## Loading of mobile web templates

The objective was pretty simple. We did not want to manually specify mobile templates at all. It means that in controllers, we wanted the same feature we already had: being able to ask for a template name, and the real loaded template must be the web mobile one if we are in a web mobile context.

But we wanted to have more. Any time we use twig `render()`, `include()` or any function loading a template, we wanted the same behavior: we wanted it to load the mobile version if it exists. Because let's say, you are on a mobile template, you want to include a template A which is the same for web and web mobile and only exist in one generic version. And in template A, you want to include a template B, which does exist in two versions: web and web mobile. It would be very helpful if the right version of template B could be renderer automatically without having to save or get the information on whether we are in a mobile context or not.

## Altering the rendering of templates

Twig has a wonderful template inheritance system which feels under-used in our codebase. Instead of conditionally include different templates we wanted to make use of this inheritance system. Instead of `if` combined with `include`, we could define blocks, and override them.
Moving from this:

<pre>
                                                                       +-------------------------------+
                                                             include   |                               |
                                                                +------> onboardPaymentForm.html.twig  |
+-------------------+               +--------------------+      |      |                               |
|                   |     extends   |                    |      |      +-------------------------------+
| layout.html.twig  <---------------+ purchase.html.twig +------+
|                   |               |                    |      |      +-------------------------------+
+-------------------+               +---------^----------+      |      |                               |
                                              |                 +------> onlinePaymentForm.html.twig  |
                                              |              include   |                               |
                                              |                        +-------------------------------+
                                              |
                                              +
                                      rendered template
</pre>

To this:
<pre>
                                                                       +----------------------------+
                                                        extends        |                            |
                                               +-----------------------+ purchase.onboard.html.twig <----------+ rendered template
                                               |                       |                            |
+-------------------+               +----------v---------+             +----------------------------+
|                   |     extends   |                    |
| layout.html.twig  <---------------+ purchase.html.twig |
|                   |               |                    |
+-------------------+               +----------^---------+             +----------------------------+
                                               |                       |                            |
                                               +-----------------------+ purchase.online.html.twig  <----------+ rendered template
                                                        extends        |                            |
                                                                       +----------------------------+
</pre>

## Removing translation keys suffixes

Actually, solving this one was easy, because the solution came from the previous case. If we had now have two different templates, `purchase.online.html.twig` and `purchase.onboard.html.twig`,  we do not need suffixes anymore. We can just write explicit onboard translation keys in the onboard and online translation keys in the online template.

# The TECH part

## Twig Engine and Twig Environment

Because `\Twig_LoaderInterface` does not grant access to any context, but only to the template name, we were not able to make a loader aware of some context.
Instead, we added a `TwigEnvironment` and a `TwigEngine` objects.

`TwigEnvironment` extends `\Twig_Environment` from Twig directly, and `TwigEngine` extends `Symfony\Bundle\FrameworkBundle\Templating\TemplateReference\TwigEngine`.

In `TwigEnvironment` we overrode the `loadTemplate()` method to add a new parameter: `array $context = []`.

We also overrode all methods calling `loadTemplate()` to make them give the context to it. These methods are `render()`, `display()`.
And last, we added a new dependency : a `NameResolver` object. When `TwigEnvironment` has a `NameResolver` set, it will ask it to resolve the correct template name, given a template name and the context that was given. The `NameResolver` will have a chance to change the template name to load, and the classical execution flow will continue: the twig loader will load the template, and it will be rendered.

In `TwigEngine`, we overrode rendering methods like `render()` and `stream()` to make them give the context when they call rendering methods on the `TwigEnvironment` object.

## NameResolver

The `NameResolver` job is to change the template name you asked to load, by using the context you provide and some configuration.

For example, you ask to render the `purchase.html.twig` template. This template is configured to have some variants. The first one is online booking vs. onboard booking, and the second one is web vs. mobile web.

If the context specifies it's an onboard trip, and we are on the mobile web platform, then it will change the template name to `purchase.onboard.mobi.html.twig`.
If the context specifies it's an online trip and we are on the web platform, then it will change the template name to `purchase.online.html.twig`.

To be able to transform the template name, the `NameResolver` uses `Matcher` objects.
The configuration allows us to specify, for any given template, what matchers must try to match the context with the real template name that should be loaded. It also allow us to keep the system fully backward compatible. Only configured template will be processed by the `NameResolver`. Therefore you can migrate your templates progressively to this new system and be sure any other template will stay as it was before.

## Matchers

The job of `Matcher` objects is simple. They try to match, either in the context given when rendering a template, either using other services if we are in a given situation and we should modify the template name to render. The two first matchers available are:

- `BookingTypeMatcher` : This one expects you to provide the booking type used. You have to provide it in the `_booking_type` key of the context array, and it can be either a `BookingType` object, or `BookingType::BOOKING_*` constant. Depending on this, it will return either `'online'` or `'onboard'`, meaning the `NameResolver` will have to add this fragment to the template name.
- `MobileVersionMatcher` : This one uses the `BlablacarContext` object to determine whether or not we are on the mobile web version. If yes, then it will return `'mobi'`. If not, it will return false, meaning we do not have to change the template name.

If both matchers matches, then the `NameResolver` will convert `purchase.html.twig` to `purchase.online.mobi.html.twig` for example.

Matchers have a priority. It is used by the `NameResolver` to know in which order the new template name should be written. Because the `BookingTypeMatcher` has a higher priority than the `MobileVersionMatcher`, the `NameResolver` will produce `purchase.online.mobi.html.twig` and not `purchase.mobi.online.html.twig`.

## Usage

The configuration is organized this way:
The `app/config/views.yml` file is an entry point to imports all other view config files.
`app/config/views/*.yml` are the files containing the configuration for the template selection. For now, we created one file per symfony bundle using this template loading system.

Template configuration is organized by groups. The main point of groups is to group together templates needing the same matchers, thus avoiding some repetition during the configuration.

In the group configuration you can specify which matchers will be applied to all the templates contained in this group.
Then you need to list the templates contained in the group, by using their names (See Template names section bellow). If you have, for one specific template in a group, the need to add an additional matcher to it, you can apply a specific matcher to this template without having to create a new group specifically for it.

## Template names

To be able to quickly spot in the codebase what template is handled by this template loading system, we introduced a new syntax for template names. The format is the following:
<BundleName>:<PathToTemplateWithExtension>

Some examples:

- Main:Homepage/homepage.html.twig
- Booking:Booking/purchase.html.twig

This syntax was introduced to solve an issue with the default symfony notation using two ':' characters. When you have many subfolders, you have many ways to reference a template name, for example:

- BlablacarBookingBundle:Emails/Passenger:drvr_cancel_all.html.twig
- BlablacarBookingBundle:Emails:Passenger/drvr_cancel_all.html.twig
- BlablacarBookingBundle::Emails/Passenger/drvr_cancel_all.html.twig

All of these are valid template names that will load the same template in Symfony. But since we need to add the template name in the configuration, we don't want to have to write all the possibilities, and we don't want people to get confused because they wrote the configuration in a way and call the template in code in another way, we chose to make this new naming system, removing one ':' character, making a unique notation to resolve templates.

The previous example can only be written in the following form with our naming:

- Booking:Emails/Passenger/drvr_cancel_all.html.twig

## Configuration reference

<pre>
blablacar_main:
    # Configuration of the views
    views:
        # Prototype
        group_name:
            # Matchers to add to all templates of this group
            matchers:             []
            # Templates contained in this group
            templates:
                # Prototype
                template_name:
                    # Additional matchers for this template
                    matchers:             []
</pre>