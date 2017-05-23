# Puppet Local Group Policy (Windows)

**NOTE:** This module has been abandoned both upstream and here. You are 
strongly advise to *not* use this module nor the upstream version.

I have personally given up trying to use this local_group_policy module. The
main reason is that it breaks if there are any group policy localization files.
This happens if one has installed localized software which bundles group
policies with it. This renders to provider useless except in pure American
English environment.

What the provider essentially does (via tons of detours) is change the contents
of Registry.pol, which is a simple text file with contents such as this
(linefeeds added for clarity):

    PReg
    [Software\Microsoft\Windows\CurrentVersion\Policies\System;Wallpaper;;F;c:\users\samuli\pictures\green.jpg]
    [Software\Microsoft\Windows\CurrentVersion\Policies\System;WallpaperStyle;;;4]
    [Software\Policies\Microsoft\Windows\ControlPanel\Desktop;ScreenSaveActive;;;1]

Serving the Registry.pol file from the Puppet fileserver, or generating it from 
a template, is much simpler and more robust approach than what the 
local_group_policy used. Here's a snippet of Puppet code that refreshes the 
local group policy:

    file { 'local-group-policy':
        ensure => 'present',
        name   => 'C:\Windows\System32\GroupPolicy\User\Registry.pol',
        source => 'puppet:///modules/windesktop/Registry.pol',
        notify => Exec['refresh-group-policy'],
    }
    
    exec { 'refresh-group-policy':
        command     => 'c:\Windows\System32\gpupdate.exe /force',
        refreshonly => true,
    }

The Local Group Policy Editor (GUI) can be used to figure out what to put into
Registry.pol.
