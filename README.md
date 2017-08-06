<h1 align="center">
  Feedback
  <br>
</h1>

<h4 align="center">A framework to measure an application's usability</h4>

# How to install
### via `Carthage`
<br>
1. Navigate to your project's directory

    ~~~~
    $ cd /Users/{alias}/path/to/project/
    ~~~~
<br>
2. Create a Cartfile

    ~~~~
    $ touch Cartfile
    ~~~~
<br>
3. Update the Cartfile with the location of Feedback
    - Atom is the preferred editor for the Cartfile. If you use another editor, make sure the double quotes surrounding the branch name (e.g. "master") and the url are of the right format.
    - Inconsistencies in other editors with formatting generally occurs when you directly copy and paste the entire line (see `Cartfile` below) in the Cartfile.
        - If you copy and paste the whole line, simply delete the double quotes one by one and type them in manually.<br><br>
    
    `Cartfile`
    ~~~~
    # Development and Test Dependencies
    
    git "git@sfgitlab.opr.statefarm.org:ios-packages/Feedback.git" "master"
    ~~~~
<br>
4. Run `carthage update --no-build`
    - This will fetch dependencies into a Carthage/Checkouts folder and build each one or download a pre-compiled framework.
    - Make sure you're in the same directory as your Cartfile.

        ~~~~
        $ pwd
        /Users/{alias}/path/to/project/
        ~~~~

    ~~~~
    $ carthage update --no-build
    ~~~~
<br>
5. Create a workspace (**Skip to step 6 if you already have a workspace**)
    1. In your project, create the workspace.<br><br>
    ![File -> New -> Workspace...](img/creating_a_workspace_1.png)<br><br>
    2. Close out of your existing project while keeping the workspace open.
        - From now on, perform all work in `.xcworkspace` and not `.xcodeproj`.<br><br>
    3. Drag and drop your previous project into the workspace.<br><br>
    ![Dragging and dropping existing .xcodeproj into new .xcworkspace](img/creating_a_workspace_2.png)
<br><br>
6. In the Build Settings of your project in the `.xcworkspace`, set `Enable Bitcode` to **No**<br><br>
  ![Disabling Bitcode in Build Settings](img/disable_bitcode.png)
<br><br>
7. Drag `Feedback.xcodeproj` into your `.xcworkspace`
    - `Feedback.xcodeproj` will be located in `~/Carthage/Checkouts/Feedback/Feedback/`<br><br>
    ![Including Feedback.xcodeproj into existing .xcworkspace](img/including_feedback_xcodeproj.png)
<br><br>
8. Create a new group named `Frameworks` (if it doesn't already exist in your project)<br><br>
    ![Including Feedback.xcodeproj into existing .xcworkspace](img/new_group.png)<br><br>
9. Add Feedback as an `Embedded Binary`
    - `Embedded Binaries` is located under the `General` tab in your project target<br><br>
    ![](img/embedded_binary.png)
<br><br>
11. Drag `Feedback.framework` into `Frameworks`<br><br>
    ![](img/dragging_into_group.png)<br><br>
12. Import Feedback into your `Swift` file(s)
    - Note: If you encounter the compiler error: `No such module 'Feedback'`, build your project.<br><br>
    ![](img/importing_feedback.png)

<br><br>
# Setting the Data Points

- In `viewDidLoad()` within your desired `ViewController`, set the Data Points by adding them to the "main" environment.
    - The "main" environment can be thought of as your default set of data points. However, if your application needs more than one type of feedback, other environments can be created.

~~~~
override func viewDidLoad() {
    super.viewDidLoad()
        
    FeedbackEnvironment.add(with: [Emoji.bad, Emoji.ok, Emoji.good])
}
~~~~

<br><br>
# Presenting and Handing the Callbacks

~~~~
@IBAction func showFeedback(_ sender: UIButton) {
    // 1.
    present()

    // 2.
    _ = FeedbackEnvironment.main.vc?.onSubmit { (choice, text, completion) in
        _ = "Your app is \(String(describing: choice))"
        _ = "The feedback message: \(String(describing: text))"

        completion()
    }

    // 3.
    _ = FeedbackEnvironment.main.vc?.onCancel {
        self.dismiss(animated: true, completion: nil)
    }
}
~~~~

1. This function presents `FeedbackViewController`.
2. `onSubmit` is a Closure that is called when a user "sends" feedback. Three things are available to you at this time.<br><br>
  1. `choice`: The currently selected data point.
  2. `text`: The text the user has entered in the `UITextView`.
  3. `completion`: Performs a segue to the `FeedbackCompletionViewController`.<br><br>
3. `onCancel` is a Closure that is called when a user "cancels" out of feedback. It is up to you how you want to handle this action.

<br><br>
# Creating Custom Data Points

All work must be done in the `master` branch of Feedback for your changes to persist.

1. Navigate to `Feedback/Feedback/Models/DataPoints.swift`.
2. Create your enumeration (this must conform to FeedbackChoiceProtocol).

<br><br>
### FeedbackChoiceProtocol

~~~~
public protocol FeedbackChoiceProtocol {
    var image: UIImage? { get }
    var title: String? { get }
}
~~~~

<br>
### CustomDataPoint
~~~~
public enum CustomDataPoint: FeedbackChoiceProtocol {
    // 1.
    case bad
    case good
    case great

    // 2.
    public var title: String? { return nil }

    // 3.
    public var image: UIImage? {

        switch self {
        case .bad:
            return UIImage(named: "bad", in: Bundle(for: FeedbackViewController.self), compatibleWith: nil)
        case .ok:
            return UIImage(named: "good", in: Bundle(for: FeedbackViewController.self), compatibleWith: nil)
        case .good:
            return UIImage(named: "great", in: Bundle(for: FeedbackViewController.self), compatibleWith: nil)
        }
    }
}
~~~~

1. These are the `choice`s used to measure your application's usability.
2. **Note: As of July 27, 2017**, setting the title has not been tested and is still needs to be developed. The purpose of `title` was if you wanted to set a literal character for a data point (e.g. "😊 ").
    - In other words, **only set the `image` property when creating custom data points**.
3. You must set an image that will be mapped to each case.
