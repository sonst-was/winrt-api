---
-api-id: E:Windows.UI.Xaml.FrameworkElement.Loaded
-api-type: winrt event
---

<!-- Event syntax
public event Windows.UI.Xaml.RoutedEventHandler Loaded
-->

# Windows.UI.Xaml.FrameworkElement.Loaded

## -description
Occurs when a [FrameworkElement](frameworkelement.md) has been constructed and added to the object tree, and is ready for interaction.

## -xaml-syntax
```xaml
<frameworkElement Loaded="eventhandler"/>
 
```


## -remarks
Although this event uses the [RoutedEventHandler](routedeventhandler.md) delegate and [RoutedEventArgs](routedeventargs.md) as event data, the event is not a routed event. It can be handled only on the element that originates the event (in other words, the *sender*). [OriginalSource](routedeventargs_originalsource.md) in event data for this event is always **null**.

### Loaded and object lifetime

In the Windows Runtime implementation, the [Loaded](frameworkelement_loaded.md) event is guaranteed to occur after a control template is applied, and you can obtain references to objects that are created by applying the XAML template.

The [Loaded](frameworkelement_loaded.md) event can be used as a point to hook up event handlers on elements that come from a template, or to invoke logic that relies on the existence of child elements that are the result of an applied template. [Loaded](frameworkelement_loaded.md) is the preferred object lifetime event for manipulating element tree structures with your app code prior to the display of XAML controls for your UI. It is also appropriate to call the [VisualStateManager.GoToState](visualstatemanager_gotostate.md) method from a [Loaded](frameworkelement_loaded.md) handler in order to set an initial view state that is defined in the template, if there's no other event that also occurs on initial layout ([SizeChanged](frameworkelement_sizechanged.md) does occur on initial layout).

The timing of [Loaded](frameworkelement_loaded.md) in the Windows Runtime implementation is similar to its timing in the Windows Presentation Foundation (WPF) implementation. In contrast, the Microsoft Silverlight implementation has a timing issue where you cannot rely on the template being loaded when [Loaded](frameworkelement_loaded.md) occurred. If you are migrating XAML or code-behind from these XAML frameworks, you may want to adjust what you do in a [Loaded](frameworkelement_loaded.md) handler to be appropriate for the template-load timing of the Windows Runtime implementation.

To access the items that come from an applied template, you can use the [VisualTreeHelper](../windows.ui.xaml.media/visualtreehelper.md) static methods and navigate child elements by index. Or you can call the [FindName](frameworkelement_findname.md) method on the root element of the templated content to find a specific part of the template with a given [x:Name attribute](http://msdn.microsoft.com/library/4ff1f3ed-903a-4305-b2bd-dcd29e0c9e6d) value. Note that you must call [FindName](frameworkelement_findname.md) on the template root rather than the control itself, because there is a XAML namescope created any time that objects are created by a template that is specific to that template (for more info, see [XAML namescopes](http://msdn.microsoft.com/library/eb060cbd-a589-475e-b83d-b24068b54c21)). To get to the template root, use `VisualTreeHelper.GetChild(target,0)` where `target` is the object where the template is applied. Once you've got that root you can get to the named parts thereafter.

If you are deriving from an existing control, instead of handling [Loaded](frameworkelement_loaded.md) on a per instance basis, you can override [OnApplyTemplate](frameworkelement_onapplytemplate.md) to make the behavior part of the default class behavior. [OnApplyTemplate](frameworkelement_onapplytemplate.md) is specifically intended as the callback for this situation, where you have a tree of objects from the applied template and now you want to examine or adjust the visuals. This is a key part of defining behavior for a custom control, including actions such as declaring the starting visual states and wiring class handlers that can't be defined using the **On*** override pattern. One difference is that from the [OnApplyTemplate](frameworkelement_onapplytemplate.md) scope you should use [GetTemplateChild](../windows.ui.xaml.controls/control_gettemplatechild.md) to find named parts rather than [FindName](frameworkelement_findname.md).

[LayoutUpdated](frameworkelement_layoutupdated.md) is a related event. The [LayoutUpdated](frameworkelement_layoutupdated.md) event is the last "object lifetime" event in the sequence of enabling a control, and occurs after [Loaded](frameworkelement_loaded.md). However, [LayoutUpdated](frameworkelement_layoutupdated.md) is fired for objects that are involved in a layout change, not just successive parents in the tree. Several objects in a UI might all fire [LayoutUpdated](frameworkelement_layoutupdated.md) at the same time. Layout changes happen for a variety of reasons, such as the user changing the view state or screen resolution, or programmatic resize of other elements in the same UI or layout container. For this reason, [Loaded](frameworkelement_loaded.md) is usually a better choice for running code that works with an initial layout or an applied template.

For app code that uses navigation between pages, do not use [Page.OnNavigatedTo](../windows.ui.xaml.controls/page_onnavigatedto.md) for element manipulation or state change of controls on the destination page. The [OnNavigatedTo](../windows.ui.xaml.controls/page_onnavigatedto.md) virtual method is invoked before the template is loaded, thus elements from templates aren't available yet. Instead, attach a [Loaded](frameworkelement_loaded.md) event handler at the root of the newly loaded page's content, and perform any element manipulations, state changes, event wiring and so on in the [Loaded](frameworkelement_loaded.md) event handler.

## -examples
Handlers for [Loaded](frameworkelement_loaded.md) and [Unloaded](frameworkelement_unloaded.md) are automatically attached to any page that uses the **NavigationHelper** class from the project templates for support. The event wiring is done in the constructor. The handler is written using a lambda, and attaches other event handlers so that page navigation can use mouse or keyboard events.

```csharp
            this.Page.Loaded += (sender, e) =>
            {
                // Keyboard and mouse navigation only apply when occupying the entire window
                if (this.Page.ActualHeight == Window.Current.Bounds.Height &&
                    this.Page.ActualWidth == Window.Current.Bounds.Width)
                {
                    // Listen to the window directly so focus isn't required
                    Window.Current.CoreWindow.Dispatcher.AcceleratorKeyActivated +=
                        CoreDispatcher_AcceleratorKeyActivated;
                    Window.Current.CoreWindow.PointerPressed +=
                        this.CoreWindow_PointerPressed;
                }
            };
```

The [Loaded](frameworkelement_loaded.md) event is a good time to start decorative animations that aren't tied to theme animations or other triggers. This example shows triggering a [PointAnimation](../windows.ui.xaml.media.animation/pointanimation.md) in XAML, by wiring a [Loaded](frameworkelement_loaded.md) handler to a method that calls [Begin](../windows.ui.xaml.media.animation/storyboard_begin.md) on an animation [Storyboard](../windows.ui.xaml.media.animation/storyboard.md).



[!code-cpp[Pointanimation_cs](../windows.ui.xaml/code/pointanimation/cpp/Page.xaml.cpp#SnippetPointanimation_cs)]

[!code-xml[Pointanimation](../windows.ui.xaml/code/pointanimation/csharp/Page.xaml#SnippetPointanimation)]

[!code-vb[Pointanimation_cs](../windows.ui.xaml/code/pointanimation/vbnet/Page.xaml.vb#SnippetPointanimation_cs)]

[!code-xml[SnippetPointanimationusingkeyframes](../windows.ui.xaml/code/pointanimationusingkeyframes/csharp/Page.xaml#SnippetPointanimationusingkeyframes)]

[!code-vb[Pointanimationusingkeyframes_cs](../windows.ui.xaml/code/pointanimationusingkeyframes/vbnet/Page.xaml.vb#SnippetPointanimationusingkeyframes_cs)]

[!code-cpp[Pointanimation_cs](../windows.ui.xaml/code/pointanimation/cpp/Page.xaml.cpp#SnippetPointanimation_cs)]

[!code-csharp[Pointanimation_cs](../windows.ui.xaml/code/pointanimation/csharp/Page.xaml.cs#SnippetPointanimation_cs)]

[!code-vb[Pointanimation_cs](../windows.ui.xaml/code/pointanimation/vbnet/Page.xaml.vb#SnippetPointanimation_cs)]

## -see-also
[OnApplyTemplate](frameworkelement_onapplytemplate.md), [VisualTreeHelper](../windows.ui.xaml.media/visualtreehelper.md), [Control.Template](../windows.ui.xaml.controls/control_template.md), [Events and routed events overview](http://msdn.microsoft.com/library/34c219e8-3efb-45bc-8bbd-6fd937698832), [Quickstart: control templates](http://msdn.microsoft.com/library/67c424ae-afb1-4560-a6a8-4a3506775d77)
