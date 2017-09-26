---
layout:     post
title:      Get-VesterTest Changes
date:       2017-09-01 15:00:00
author:     Robin Haberstroh
summary:    Updating Get-VesterTest to include a custom view
categories: Blog
tags:
 - Vester
 - PowerShell
 - GitHub
---



# Introduction

[@brianbunke] recently released v1.2.0 of [Vester] to the community and with that release he included the `Get-VesterTest` command which parses the tests directory and outputs the Name, Scope, and Description of the tests. 

## Background

My involvement with Vester so far has largely been related to configuring tests for the [Security Hardening Guide] remediation steps which inclue the recommended values. To date I've tried to include those recommendations either through commented lines or in the `$Description` variable. With the new command (`Get-VesterTest`) I saw an opportunity to revisit my tests and add a `$Recommendation` value and update the `Get-VesterTest` command to include that new property.

## Steps taken

On the flight back from VMworld2017 I thought I would make use of my time instead of watching _30 Rock_ for the next 3.5 hours so I fired up [VS Code] and started digging through Vester to see how `Get-VesterTest` worked.

It turns out there is a private function named `Extract-VestDetails.ps1` (uh-oh, "Extract" isn't listed as an accepted verb in `get-verb` <i class="fa fa-frown-o" aria-hidden="true"></i> ) that is doing the work that I was interested in. Within that function is a `PSCustomObject` that contained all of the values. Simply inserting `Recommendation = $Recommendation` after the Description value was all I needed to do. I updated one of my tests with that new variable and ran my test.

On the first run I got the Name, Scope, and Description. No worries, it's common practice to only display key values on the initial run. I ran the command again (up arrow FTW!) and piped it through to `format-list`. Hmm, Name, Scope, and Description are still all that I'm getting. Thinking I did  something wrong I decided to pipe the results through to `get-member` and sure enough my new property was there along with the other items from the test files. Interesting, time to dig deeper.

The end of `Get-VesterTest` contained the following

```posh
If (-not $Simple) {
    # Reduce default property set for readability
    $TypeData = @{
        TypeName = 'Vester.Test'
        DefaultDisplayPropertySet = 'Name','Scope','Description'
    }
    # Include -Force to avoid errors after the first run
    Update-TypeData @TypeData -Force
}
```

The `DefaultDisplayPropertySet` was what was controlling the output information. As I mentioned earlier though, I'm used to the "pretty" `Format-Table` view to be limited but typically expect all data returned in the `Format-List` view. Fortunately for me, I travel with a copy of _Learn PowerShell ToolMaking in a Month of Lunches_ by [Don Jones] and quickly found the chapters discussing how to make your own custom view.

## Creating a custom view

After reading through the chapters and refreshing my memory I started the process of removing the restrictive Type `DefaultDisplayPropertySet` by commenting out the code displayed above and validating that I received the full property set. Once that was confirmed I honestly followed a little too closely what the book was trying to make me do, but finally figured out that the TestType was being defined in the aforementioned private function so I didn't need to specify it in `Get-VesterTest.ps1` file. 

With that, I started creating the `Vester.Format.ps1xml` file which can be seen below

```xml
<?xml version="1.0" encoding="utf-8" ?>

<Configuration>
    <ViewDefinitions>
        <View>
            <Name>Vester.Test</Name>
            <ViewSelectedBy>
                <TypeName>Vester.Test</TypeName>
            </ViewSelectedBy>
            <TableControl>
                <TableHeaders>
                    <TableColumnHeader/>
                    <TableColumnHeader>
                        <Width>16</Width>
                    </TableColumnHeader>
                    <TableColumnHeader/>
                </TableHeaders>
                <TableRowEntries>
                    <TableRowEntry>
                        <TableColumnItems>
                            <TableColumnItem>
                                <PropertyName>Name</PropertyName>
                            </TableColumnItem>
                            <TableColumnItem>
                                <PropertyName>Scope</PropertyName>
                            </TableColumnItem>
                            <TableColumnItem>
                                <PropertyName>Description</PropertyName>
                            </TableColumnItem>
                        </TableColumnItems>
                    </TableRowEntry>
                 </TableRowEntries>
            </TableControl>
        </View>
    </ViewDefinitions>
</Configuration>
```

My intent was that the output for the default view match what Brian started with. But now when I pass those results to `Format-List` I get all of the details which was my original intent.

Finally, I had to update the `Vester.psd1` file to reflect the new view file by uncommenting `FormatsToProcess` and adding the file name to the array.

[@BrianBunke]: http://www.twitter.com/brianbunke

[Don Jones]: http://www.twitter.com/concentrateddon

[Vester]: https://github.com/WahlNetwork/Vester

[Security Hardening Guide]: https://www.vmware.com/security/hardening-guides.html

[VS Code]: https://code.visualstudio.com/