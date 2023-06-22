---
layout: "../../layouts/BlogLayout.astro"
title: Extend bootstrap package theme tracking
date: 12/06-2023
description: Learn how to extend bootstrap package theme for TYPO3 with extendable tracking configuration
slug: typo3-bootstrap-package-extend-track
author: Leo Knudsen
---

Do you often need an easier solution to insert tracking straight from the backends constant editor in TYPO3?
worry no more

I have found a solution for extending Bootstrap Package theme for TYPO3 with more tracking configuration

the first step we are going to take is registering the constant variables to use to create your tracking scripts
lets look into Google Analytics and Facebook pixel scripts implementations

constants.typoscript
```txt
    #cat=bootstrap package: tracking/t_001/001; type=string; label=Google Analytics ID: Define your Google Analytics ID
    page.tracking.google.analytics = 

    #cat=bootstrap package: tracking/t_001/002; type=string; label=Meta pixel ID: Define the customer's Meta pixel ID 
    page.tracking.facebook.pixel = 
```

After setting tracking ID values in the constants editor
we can finally use the values to implement the expected tracking scripts

setup.typoscript
```txt
    ## Define google analytics and facebook pixel scripts if any ID values are set in constants editor
    page.headerData {
        100 = TEXT
        100.if.isTrue = {$page.tracking.google.analytics}
        100.value (
            <script async src="https://www.googletagmanager.com/gtag/js?id="'{$page.tracking.google.analytics}'"></script>
            <script>
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());

            gtag('config', '{$page.tracking.google.analytics}');
            </script>
        )

        200 = TEXT
        200.if.isTrue = {$page.tracking.facebook.pixel}
        200.value (
            <script data-cookieconsent="marketing">
            !function(f,b,e,v,n,t,s)
            {if(f.fbq)return;n=f.fbq=function(){n.callMethod?
            n.callMethod.apply(n,arguments):n.queue.push(arguments)};
            if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
            n.queue=[];t=b.createElement(e);t.async=!0;
            t.src=v;s=b.getElementsByTagName(e)[0];
            s.parentNode.insertBefore(t,s)}(window, document,'script',
            'https://connect.facebook.net/en_US/fbevents.js');
            fbq('init', '{$page.tracking.facebook.pixel}}');
            fbq('track', 'PageView');
            </script>
            <noscript><img height="1" width="1" style="display:none"
            src=https://www.facebook.com/tr?id={$page.tracking.facebook.pixel}&ev=PageView&noscript=1
            /></noscript>
        )
    }
```