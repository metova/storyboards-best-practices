# Preamble

Since the introduction of storyboards, Metova’s iOS developers have debated the advantages and disadvantages of them. Some developers think a storyboard’s easy-to-understand organizational structure outweigh any drawbacks. Others believe the drawbacks, such as lack of support for initializer dependency injection, make them cumbersome to use and cause developers to compromise on code clarity . Whichever side of the argument our developers fall on, however, they all agree that, when using storyboards, the practices outlined in this document represent the optimal strategies to maximize storyboards’ features while minimizing their drawbacks.

# Benefits

There are clearly some important benefits to using storyboards that contribute to their popularity amongst developers. Broadly speaking, all these benefits revolve around the visual nature of storyboards in providing important architectural and flow information in a snapshot, saving developers precious time constructing those representations in their minds by reviewing the underlying code. Having said that, there are also some non-visual benefits that arise from the use of storyboards worth mentioning here as well.

#### Navigation Stacks

It is almost effortless to see the relationship of various views with one another, how they fit into a navigation stack and which views are managed by which navigation controllers, when a project uses storyboards. This benefit is especially tangible for anyone joining a large project for the first time. In a matter of seconds or minutes, the newcomer is up-to-speed with how navigation stacks are set up.

#### Segue And User Flow

Segues are strictly storyboard elements that provide a prominent visual reference to the navigational direction within the app and help further clarify the UI flow of a project. Together with navigation stacks, segues can help new collaborators on a project quickly learn how the project is organized and how the decision-tree maps onto the user flow in the app—where a user would end up in the app by following one set of decisions vs. another.

#### Organizational Structure

Because storyboards can house multiple view controllers, they can create an organizational structure for the app. View controllers that are part of the same component of the app can be put in the same storyboard allowing for easy separation of concerns.

# Drawbacks

There are some aspects to storyboards that are not ideal. The following section points out the most obstructing problems with storyboards and offers solutions or workarounds.

#### Potential Problem: Storyboards get too large to manage effectively among a large team of developers.

Having multiple people work on a single storyboard is definitely not a great way to work. Merge conflicts inevitably result and precious time is wasted trying to sort them out. In order for storyboards to be effective for a team of developers, it is best to break up the app by components and assign a storyboard for each component. 

For instance, a team could have an “Onboarding” storyboard for login and sign-up screens, “Home” for the app’s home screen, and “Settings” for the app’s settings screens. This would allow developers to work on each component of the app while not trampling over each other’s work in the storyboard. Each storyboard should contain only as many view controllers as needed for its particular component; the smaller the storyboard, the better. In order to allow for easy segues between these component based storyboards, storyboard references should be used. This allows the use of segues without needing to put all segue connected view controllers into one storyboard. 

#### Potential Problem: Storyboard IDs for view controllers are strings, which is bad to use in code due to the potential of typos. Furthermore, string literals are not checked at compile time, so the potential for error when attempting to access a view controller is high.

The easiest way to ensure that your storyboard IDs used in code are compile time checked is to use a tool like [SwiftGen](https://github.com/SwiftGen/SwiftGen). SwiftGen is a tool that creates enums for various assets in an xcode project. Relevant to our discussion, it creates enums for each storyboard and view controller. This eliminates the potential errors of using strings to identify them. The SwiftGen script can be run on every build, so you never have to manually edit the enums as new storyboards and view controllers are added.

#### Potential Problem: Segues are not flexible enough. What if I want to use one segue that originates from two different UI elements?

This problem is solved by attaching the segue to the view controller rather than specific UI elements. Then, you can call `performSegue(withIdentifier: sender:)` from anywhere in the view controller to perform the segue. This makes segues very flexible and easy to use. 

#### Potential Problem: Storyboards don’t allow for injection upon instantiation of a view controller

To see a full discussion on this issue, please see: Storyboards and Dependency Injections.
While there is no direct solution to this problem with storyboards, the best workaround is to create your own injection method that an instantiator calls to pass objects to your view controller. This will allow you to use implicitly unwrapped optionals for properties that are necessary for your view controller while providing an interface with which a presenting view controller can interact. While having implicitly unwrapped optional properties is still not as pure as having non-optional properties on the view controller, combining them with an injection method will insure that necessary properties are assigned before use. If an attempt to present the controller occurs without calling the injection method, the developer will know on their first run after implementation because the app will crash when the screen is presented.  The following example code shows how such an injection method should be set up.

```swift
class UserProfileViewController: UIViewController {

	private var user: User!

	func injectDependencies(user: User) {

		self.user = user
	}

	override func viewDidLoad() {

		super.viewDidLoad()

		assertDependenciesHaveBeenInjected(dependencies: [user])
	}
}

extension UIViewController {

	func assertDependenciesHaveBeenInjected(dependencies: [Any?]) {

		for dependency in dependencies {

			guard let _ = dependency else {

				assertionFailure("Missing injection in \(type(of: self))")
				return
			}
		}
	}
}
```