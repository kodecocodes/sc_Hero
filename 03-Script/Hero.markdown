Hey, what's up everybody, this is [Jessy|Catie]. In this screencast we're going to introduce you to the world of elegant transitions using the insanely popular Hero framework from Luke Zhao.

Hero is a framework for creating custom view controller transitions without having to learn Apple's cumbersome transition APIs—you won't find any mention of `UIViewControllerAnimatedTransitioning` or `UIViewControllerContextTransitioning` in this screencast.

_Co-presenter, looking annoyed_: But [Jessy|Catie], you _just did_ mention them!

Hero is built around the concept of Keynote's Magic Move feature—you simply create relationships between source and destination views and then Hero will automagically transition them from their old state to their new state. But it's not limited to just matched views, as you can also provide custom animations for unmatched views and these'll run in parallel to those handled automatically.

But enough with the pleasantries! The easiest—and most enjoyable!—way to understand Hero is to begin using it, so let's dive straight in.

> Open Podfile

I've gone ahead and installed `Hero` using Cocoapods, and will therefore be working in the Xcode workspace rather than the project.

> Open Betamax.xcworkspace, and then build and run using the iPhone 6S (or Plus) simulator

You'll notice that I've launched the app using the iPhone 6S simulator—this is because of a bug with the iPhone 7 simulators that affects view snapshotting. There are more details available in the README that accompanies Hero, but the TL;DR is that, at least for now, it's best to use one of the 6S simulators or a real device.

The sample I'll be using throughout this screencast is a fictional app for viewing raywenderlich.com videos... 

_Co-presenter, looking inquisitively_: Hmmm, fictional you say?

Yes, fictional. But who knows what's around the corner?

Now, back to what I was saying. The set up is a pretty standard affair—I have a list of videos backed by a table view, and tapping one takes me to the video detail view, with the whole thing wrapped up inside a navigation controller.

To make the transition between the two views more engaging, I would like, when the user taps a video, for the thumbnail to persist during the transition, scale up, and even switch to a more detailed banner, before settling into place in the detail view. This is _exactly_ the kind of thing Hero was built for, and which it allows me to achieve with very little effort.

The first thing I need to do is to enable Hero on each of the view controllers involved in the transition...

> Open Main.storyboard, and then select Navigation Controller in the project outline

As I'm using a navigation controller I only have to enable Hero on that controller and each of the child view controllers will inherit the setting. Without the navigation controller I'd have to go through and set each one manually.

> With the navigation controller selected, jump to the Attributes inspector, find the View Controller section, and set Is Hero Enabled to On

And that's all there is to enabling Hero! The setting is also available in code via the `isHeroEnabled` property, which is added to any `UIViewController` subclass once you've imported `Hero`. Speaking of which...

> Open PlaylistViewController.swift and add the following `import` statement

```
import Hero
```

Hero works by taking a snapshot of the source and destination views prior to the transition commencing, and then animating any views shared between the two to ahcieve the elegant and engaging transitions. Hero provides four different options for handling the snapshot, which vary in their accuracy and complexity—the default is fine for most circumstances, but will struggle with custom views, views with masks, or views with complex heirarchies. 

As both my source and destination views have quite complex layouts, I'll instruct Hero to use one of the slower but more accurate snapshotting techniques.

> Add the following to the bottom of `viewDidLoad()`

```
view.heroModifiers = [.useLayerRenderSnapshot]
```

Hero uses the idea of modifiers—it provides a set of basic functionality, and then you provide an array of modifiers to any view in the transition where you want to tweak or change their behaviour. 

Behind the scenes this type uses `CALayer`'s `render(in:)` method to provide a more accurate snapshot than the default one, which relies on the higher level `UIView` snapshot methods. A modifier is simply a type method or property defined on `HeroModifier`—there are all sorts of modifiers you can use, so I highly recommend you check out HeroModifier.swift in the frameworks source to see what's available.

I need to make sure both the source and the destination views are using this technique, so I'll make the same changes to VideoViewController.swift.

> Open VideoViewController.swift and add the following `import` statement

```
import Hero
```

> Add the following to the bottom of `viewDidLoad()`

```
view.heroModifiers = [.useLayerRenderSnapshot]
```

In order for Hero to know which views to animate from the source to the destination, it adds a `heroID` property to `UIView`, and therefore anything descending from `UIView`. To create a relationship between a view in the source of the transition and a view in the destination, I just need to set the `heroID` property on both views to the same string value.

As the data shown in the detail view is dependent on which table view cell is selected, the value I use for the `heroID` needs to be dynamic, and derived from the model object.

> Add the following to `configureVideo()`, just below where the image is being set on `bannerImageView`

```
bannerImageView.heroID = "\(video.id)-image"
```

The view controller is passed a `Video` model object when the segue between the two controllers is triggered. This method uses that `Video` instance to configure the necessary views. The `id` of each `Video` is unique so I use that to generate the `heroID`, thereby creating the relationship between the large image view in the detail view, and thumbnail image view of the selected table view cell.

Now I just need to configure the other side of that relationship in the custom cell class.

> Open VideoViewController.swift and, in the `didSet` observer of `video`, add the following just below where the image is being set on `thumbnailImageView`

```
thumbnailImageView.heroID = "\(video.id)-image"
```

The cell is passed an instance of `Video` just before it's returned from the table view delegate, so I can use the `id` property in exactly the same way as I did earlier. 

With the snapshotting configured, and the relationship between the two views created, I can now build and run and see the transition in action.

> Build and run, and then select a couple of different videos. One from the top, and then one from near the bottom is best to demostrate the starting position differs based on which cell is selected 

Woah! How cool is that? A single setting in Interface builder, and 4 lines of code, and I have something far more elegant and engaging than the stock push transition provided by `UINavigationController`.

However, you may have spotted a couple of issues. 

First, when returning to the table view there is jump and the content offset appears off. This is a simulator issue—if you run the app on a device it works flawlessly.

Second, the play button. As I've created a relationship between the two views being animated, Hero removes the destination view from the destination until the transtion is complete, which means the play button can be seen animating into position with the rest of the view, but underneath the banner, and then it just pops into place—this is somewhat jarring to say the least and really takes the shine off this otherwise elegant transition.

Luckily it's relatively straight forward to fix this issue—I can just use a couple of modifiers on the play button to adjust its behaviour.

> Open VideoViewController.swift

First I'll add a `didSet` property observer to `playButtonImageView`, as a convenient place to set up the modifiers.

```
didSet {

}
```

Next I'll set `heroModifiers` to an empty array temporarily.

```
playButtonImageView.heroModifiers = []
```

The easiest way to resolve this issue is to hide the play button until the transition is complete, and then show it. As I'm letting Hero manage the rest of the transition it makes sense to let it handle this as well. The framework provides a modifier that allows you to set the starting state of view if you want it to differ to what's configured in Interface Builder, so I'll use this to set the views opacity to zero, essentialy hiding it.

```
.beginWith(.opacity(Float(0)))
```

I have to instantiate the value in this way because Hero provides two overloads of `opacity()`—one for `CGFloat` and one for `Float`—but unfortunately the Swift compiler can't determine which to use, hence the use of `Float`.

Hero only takes `beginWith` modifiers into account when used alongside an animation, so I'll add the `.fade` animation modifier to the array as well. 

```
.fade
```

Usually this would animate a fade from zero to what ever alpha value has been set on the owning view. However, since I've used `beginWith` to set it to zero, the animation is effectively animating from zero to zero, thereby hiding the view all together.

> Build and run, and then select a couple of different videos.

Much better!

> Attempt to swipe back to the video playlist

Oooops! It appears as though I've stumbled across another issue that needs some attention before I can wrap up this screencast. 

`UINavigationController` by default installs a gesture recognizer that allows you to swipe back through the view controllers in its stack. However, that gesture recognizer is disabled as soon as you enable Hero on a navigation controller because Hero can't determine, at least not just yet, the progress of the swipe, which it would need in order to make the transition interactive.

But as with everything I've shown about Hero thus far, it's just a few lines of code to set things up appropriately so Hero _can_ make the transition interactive, and make you look like a 10x developer! :]

> Open VideoViewController.swift

I need to create a gesture recognizer that behaves exactly like the standard one, and luckily for me Apple provides a class—`UIScreenEdgePanGestureRecognizer`—that does just that.

> Add the following to the bottom of `viewDidLoad()`

```
let recognizer = UIScreenEdgePanGestureRecognizer(target: self, action: #selector(handlePan))
```

I'll tell the gesture recognizer to only detect touches on the left edge of the screen.

```
recognizer.edges = .left
```

And finally I  add it to the top level view in this view controller.

```
view.addGestureRecognizer(recognizer)
```

Next, I'll implement the handler for the gesture recognizer.

```
@objc private func handlePan(recognizer: UIScreenEdgePanGestureRecognizer) {
  
}
```

Gesture recognizers, at a very high level, are basically just state machines, so I'll add a switch statement with a case for each state it can be in.

```
switch recognizer.state {
case .began:
  
case .changed:
  
case .cancelled:
  
default:
  
}
```

Within the `began` state, I use a standard method on `UINavigationController` to pop the current view controller. This will kick off the transition that Hero will then handle based on the information provided by the `changed` state.

```
navigationController?.popViewController(animated: true)
```

To be able to update Hero with the progress of the gesture, I first need to know how far the user has moved their finger since the gesture began. For that I'll use `translation(in:)`, which is a method declared on `UIPangestureRecognizer` that returns the total x and y translations over the lifetime of the gesture. You can provide a view if you want to use a different coordinate system, but in this case I simply pass `nil` to use the current coordinate system.

```
let translation = recognizer.translation(in: nil)
```

> **Note**: I can't quite explain the following statement, so perhaps you'll have better luck? We need a value between 0 and 1 here, but if I simply divide `translation.x` by `view.bounds.width` the view doesn't track the finger, and the offset between the finger and the edge of the VC appears to grow. Dividing the translation value by 2 _appears_ to fix the issue, but I'm at a loss as to why?

Hero expects a value between `0` and `1` to be able to update the interactive transition, so I can calculate that by...\<INSERT EXPLANATION HERE AFTER READING THE ABOVE NOTE\>.

```
let progress = (translation.x / 2) / view.bounds.width
```

Once I have the progress, I inform Hero.

```
Hero.shared.update(progress)
```

Within the `cancelled` state, I tell Hero to cancel the transition if the gesture is cancelled.

```
Hero.shared.cancel()
```

And to simply finish the transition if the switch falls through to the `default` state.

```
Hero.shared.finish()
```

Build and run.

> Demonstrate that you can now swipe-to-go-back, and that the transition is interactive

You'll notice that the transition completes even if I cancel the gesture, which isn't a user experience we expect on iOS—I'll leave that as any exercise for you, the viewer, to fix.

Alright, that's everything we're gonna cover in this screencast.

At this point you should understand how to enable Hero on your view controllers within your storyboard, how to use Hero modifiers to both change the type of snapshot used and change the default behaviour of a specific view within a transition, and how to hook the transition up to a custom gesture recognizer so you can make it interactive.

There's a lot more to Hero that we've been able to demonstrate in this single screencast, including a plethora of different modifiers ranging from changing the position of a view to changing its rotation, and even its scale. You can also cascade modifiers over time to achieve some really nice effects. We highly recommend you check out the GitHub repo at [github.com/lkzhao/Hero](github.com/lkzhao/Hero) for more information as we have barely scratched the surface.

Thanks for watching—and Catie and I really look forward to seeing all the elegant and engaging transitions you guys build into your apps from here on out. We're out!
