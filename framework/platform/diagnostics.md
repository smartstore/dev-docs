---
description: Profiling your shop’s performance
---

# 🥚 Profiling

* Speed is important for any website
* If a website takes a long time to load, this can have adverse effects on the user experience, site traffic, and SEO.&#x20;
* Websites that are optimized for performance have an advantage over slow websites.
* Thus, it is important to speed test your shop before you launch it
* Smartstore contains a tool that can be used to measure the time from the initial request to the first byte sent to the browser (_Time to First Byte_, or short: _TTFB_): _MiniProfiler_.

## MiniProfiler

* MiniProfiler is a library and UI for profiling your application. By letting you see where your time is spent, which queries are run, and any other custom timings you want to add, MiniProfiler helps you debug issues and optimize performance.
* The default setup is for every page to display a widget on the top left corner so performance is always on your mind, like this:
* _SCREENSHOT_

### Activating MiniProfiler

* MiniProfiler is part of the **Developer Tools** module, which is installed and enabled by default.
* For more info about the MiniProfiler library, see [https://miniprofiler.com/dotnet/](https://miniprofiler.com/dotnet/).
* To activate profiling (to display the profiling widget) in the backend:&#x20;
  * Go to **Plugins / Developer Tools**
  * Turn on **Enable MiniProfiler** checkbox
* INFO: Only users with _Administrator_ role can see the profiler widget

### Exclude paths from profiling

* By default, following path prefixes are ignored by the profiler:
  * /admin/
  * /themes/
  * /bundle/
  * /media/
  * /taskscheduler/
  * /js/
  * /css/
  * /images/
* To customize this ignore list, just edit **MiniProfiler ignored paths -->** separate each entry with comma.
* e.g.: to profile backend pages also, remove the _/admin/_ entry and save the settings.

## Custom timings

* By default, the profiler widget displays timings for: controller actions, filters, view resolution & rendering, view component execution, database queries and some other app specific but important metrics.
* To hook into profiling use the `IChronometer` service to find out how much time a custom code block took to execute
* `StepStart` method starts a new timer with a key and a message that should be displayed in the widget
* `StepStop` stops the timer
* `Step` extension method makes usage even easier because it follows the disposable pattern (the disposer calls `StepStop` implicitly)
* If the **Developer Tools** module is installed and the MiniProfiler is active, the profiler widget will display your custom timing with the given message.
* INFO: If **Developer Tools** module is not installed, `NullChronometer` takes over. No exception will be thrown.

<pre class="language-csharp" data-title="Custom timing example"><code class="lang-csharp"><strong>using (_chronometer.Step("Some expensive code"))
</strong>{
    // ... execute some expensive code
}
</code></pre>

