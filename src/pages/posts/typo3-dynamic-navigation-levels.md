---
layout: "../../layouts/BlogLayout.astro"
title: Finally a fix for navigation levels
date: 12/06-2023
description: See how i managed to implement a more dynamic solution for navigation levels in bootstrap packages main navigation
slug: typo3-dynamic-navigation-levels
author: Leo Knudsen
---

Hi there, im going to show you how to implement dynamic navigation levels withins bootstrap packages main navigation

### Registering setup constants in the backend

first of all we are going to register a new constant that defines the amount of levels to allow in you're navigation

constants.typoscript
```txt
    #cat=Theme: Navigation/100; type=string; label=Define navigation levels allowed
    page.navigation.levels = 0
```

This snippet registers a constant in the constant editor

setup.typoscript
```txt
    page.settings.navigation.levels = {$page.navigation.levels} # the value of the constant
```

### Using the levels value

the main navigation partial allready provides a default submenu level

Resources/Private/Partials/Navigation/MainNavigation.html
```html
<html xmlns:f="http://typo3.org/ns/TYPO3/CMS/Fluid/ViewHelpers" xmlns:bk2k="http://typo3.org/ns/BK2K/BootstrapPackage/ViewHelpers" data-namespace-typo3-fluid="true">
<f:if condition="{mainnavigation}">
    <f:variable name="level">0</f:variable>
    <ul class="navbar-nav">
        <f:for each="{mainnavigation}" as="item">
            <f:if condition="{item.spacer}">
                <f:then>
                    </ul>
                    <ul class="navbar-nav">
                </f:then>
                <f:else>
                    <li class="nav-item{f:if(condition: item.active, then:' active')}{f:if(condition: item.icon, then:' icon-wrap')}{f:if(condition: item.children, then:' dropdown dropdown-hover')}">
                        <a href="{item.link}" id="nav-item-{item.data.uid}" class="nav-link{f:if(condition: item.children, then:' dropdown-toggle')}"{f:if(condition: '{item.target}', then: ' target="{item.target}"')}{f:if(condition: '{item.target} == "_blank"', then: ' rel="noopener noreferrer"')} title="{item.title}"{f:if(condition: item.children, then:' aria-haspopup="true" aria-expanded="false"')}>
                            <f:if condition="{theme.navigation.icon.enable} && {item.icon}">
                                <span class="nav-link-icon">
                                    <bk2k:icon icon="{item.icon}" width="{theme.navigation.icon.width}" height="{theme.navigation.icon.height}" />
                                </span>
                            </f:if>
                            <span class="nav-link-text">{item.title}<f:if condition="{item.current}"> <span class="visually-hidden">(current)</span></f:if></span>
                        </a>
                        
                        <f:if condition="{item.children}">
                            <ul class="dropdown-menu" aria-labelledby="nav-item-{item.data.uid}">
                                <f:for each="{item.children}" as="child">
                                    <f:if condition="{child.spacer}">
                                        <f:then>
                                            <li class="dropdown-divider"></li>
                                        </f:then>
                                        <f:else>
                                            <li>
                                                <a href="{child.link}" id="nav-item-{item.data.uid}" class="nav-link{f:if(condition: child.children, then:' dropdown-toggle')}"{f:if(condition: item.target, then: ' target="{item.target}"')} title="{item.title}"{f:if(condition: item.children, then:' aria-haspopup="true" aria-expanded="false"')}>
                                                <f:if condition="{theme.navigation.dropdown.icon.enable} && {child.icon}">
                                                <span class="dropdown-icon">
                                                    <f:if condition="{child.icon.0.extension} === svg">
                                                        <f:then>
                                                            <bk2k:inlineSvg image="{child.icon.0}" width="{theme.navigation.dropdown.icon.width}" height="{theme.navigation.dropdown.icon.height}" />
                                                        </f:then>
                                                        <f:else>
                                                            <f:image additionalAttributes="{loading: 'lazy'}" image="{child.icon.0}" alt="{child.icon.0.alternative}" title="{child.icon.0.title}" width="{theme.navigation.dropdown.icon.width}" height="{theme.navigation.dropdown.icon.height}" />
                                                        </f:else>
                                                    </f:if>
                                                </span>
                                                </f:if>
                                                <span class="dropdown-text">{child.title}<f:if condition="{child.current}"> <span class="sr-only">(current)</span></f:if></span>
                                                </a>
                                                <f:render section="ChildNavigation" arguments="{child: child, theme: theme, level: level}" />
                                            </li>
                                        </f:else>
                                    </f:if>
                                </f:for>
                            </ul>
                        </f:if>
                    </li>
                </f:else>
            </f:if>
        </f:for>
    </ul>
</f:if>
</html>
```

Note the f:render tag 
```html
    <f:render section="ChildNavigation" arguments="{child: child, theme: theme, level: level}" />
```

We are going to use this to render the section the value of the levels we defined in the backend
but how are we going to do that? 

Well we need to define the section to render
```html
<f:section name="ChildNavigation">
     <li class="nav-item{f:if(condition: child.active, then:' active')}{f:if(condition: child.icon, then:' icon-wrap')}{f:if(condition: child.children, then:' dropdown dropdown-hover')}">
        <a href="{child.link}" id="nav-item-{child.data.uid}" class="nav-link{f:if(condition: child.children, then:' dropdown-toggle')}"{f:if(condition: '{child.target}', then: ' target="{child.target}"')}{f:if(condition: '{child.target} == "_blank"', then: ' rel="noopener noreferrer"')} title="{child.title}"{f:if(condition: child.children, then:' aria-haspopup="true" aria-expanded="false"')}>
            <f:if condition="{theme.navigation.icon.enable} && {child.icon}">
                <span class="nav-link-icon">
                    <bk2k:icon icon="{child.icon}" width="{theme.navigation.icon.width}" height="{theme.navigation.icon.height}" />
                </span>
            </f:if> 
        </a>
        
        <f:if condition="{child.children}">
            <f:variable name="level">{level + 1}</f:variable>
            <f:if condition="{level} < {settings.navigation.levels}">
                <ul class="dropdown-menu" aria-labelledby="nav-item-{child.data.uid}">
                    <f:for each="{child.children}" as="child">
                        <f:if condition="{child.spacer}">
                            <f:then>
                                <li class="dropdown-divider"></li>
                            </f:then>
                            <f:else>
                                <a href="{child.link}" id="nav-item-{item.data.uid}" class="nav-link{f:if(condition: child.children, then:' dropdown-toggle')}"{f:if(condition: item.target, then: ' target="{item.target}"')} title="{item.title}"{f:if(condition: item.children, then:' aria-haspopup="true" aria-expanded="false"')}>
                                <f:if condition="{theme.navigation.dropdown.icon.enable} && {child.icon}">
                                <span class="dropdown-icon">
                                    <f:if condition="{child.icon.0.extension} === svg">
                                        <f:then>
                                            <bk2k:inlineSvg image="{child.icon.0}" width="{theme.navigation.dropdown.icon.width}" height="{theme.navigation.dropdown.icon.height}" />
                                        </f:then>
                                        <f:else>
                                            <f:image additionalAttributes="{loading: 'lazy'}" image="{child.icon.0}" alt="{child.icon.0.alternative}" title="{child.icon.0.title}" width="{theme.navigation.dropdown.icon.width}" height="{theme.navigation.dropdown.icon.height}" />
                                        </f:else>
                                    </f:if>
                                </span>
                                </f:if>
                                <span class="dropdown-text">{child.title}<f:if condition="{child.current}"> <span class="sr-only">(current)</span></f:if></span>
                                </a>
                                <f:render section="ChildNavigation" arguments="{child: child, theme: theme, level: level}" />
                            </f:else>
                        </f:if>
                    </f:for>
                </ul>
            </f:if>
        </f:if>
    </li>
</f:section>
```

the snippet above looks lot alike the first default defined level in the navigation partial
the only difference is that we render the same section the amount of times we defined from the levels constant we defined earlier on.

And thats all that is needed to define the amount of navigation levels
i hope this was useful to extending your applications navigation