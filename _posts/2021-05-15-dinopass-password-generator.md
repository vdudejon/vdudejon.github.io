---
layout: post
title: Powershell Function Password Generator
permalink: dinopass-generator.html
description: Some Description
date: 2021-05-15 14:04:04 -05:00
tags: "powershell security"
---

While complex passwords with random numbers and letters tend to be secure, they're not very human friendly.  In the VMware world, there are still a lot of times where you might need to enter a password by hand vs pasting it in from some password vault.  Additionally, if you aren't careful, your random password may actually be less secure than a human readable one (Ref: [Relevant XKCD](https://xkcd.com/936/))

The website https://www.dinopass.com/ is aimed at generating kid-friendly passwords, but I have found it to be a great resource for generating passwords, as long as you capitalize a letter and throw in a special character.  Convienently, they also provide a simple API.  Using the DinoPass API, I created a Powershell function to generate simple, human readable passwords that include capital and lowercase letters, numbers, and a special character.  Here is the function (also available in my [powershell github folder](https://github.com/vdudejon/Powershell):

```posh
## v 1.0 May 14 2021
## https://github.com/vdudejon/Powershell/blob/master/Get-DinoPass.psm1
## Using the api from https://dinopass.com, generates a complex but human readable password

function Get-DinoPass{
    $pass1 = (Invoke-WebRequest -Uri http://www.dinopass.com/password/simple).content
    $pass1 = (Get-Culture).TextInfo.ToTitleCase($pass1.ToLower())
    Start-Sleep -Milliseconds 200
    $pass2 = (Invoke-WebRequest -Uri http://www.dinopass.com/password/simple).content
    $pass2 = (Get-Culture).TextInfo.ToTitleCase($pass2.ToLower())
    $pass = "$pass1=$pass2"
    return $pass
}
```

Simply import the module, and you can progromatically generate complex passwords.  Perfect for rotating passwords in VMware, then updating your password safe (for example, Hashicorp Vault).

Feel free to leave suggestions (I can already think of a few changes to make), but I felt like this was a good jumping-off point.
