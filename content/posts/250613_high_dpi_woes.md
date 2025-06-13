---
title: "Against the grain"
date: 2025-06-13T08:16:00+02:00

---

Brief hack for enabling hi-DPI mode in several applications on wayland 
Desktops: 

![Screenshot visualizing grain in signal desktop compared to the rest of the UI](images/signal_desktop_grain.png)

When installing Signal Desktop on my Debian box using sway Desktop, 
I noticed ugly scaling issues on my high DPI screen. 

The solution is adding flags to the program invocation: 

```bash
--enable-features=UseOzonePlatform --ozone-platform=wayland
```

Adding these flags to my desktop file enabled the improved scaling 
behavior when running it through wofi. 

