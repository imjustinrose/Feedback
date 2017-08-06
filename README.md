<h1 align="center">
  Feedback
  <br>
</h1>

<h4 align="center">A framework to measure an application's usability</h4>
<br>
<hr>
Feedback allows developers to create their own data points for iOS applications.<br>

Snippets of code are provided below on how to use the framework. However, there is no project located in this repository because Feedback was created while I was interning at State Farm. This is a sample of the markdown file I wrote for the team.

###### © Copyright, State Farm Mutual Automobile Insurance Company, 2017.
<hr>
<br>

## Setting the Data Points

- In `viewDidLoad()` within your desired `ViewController`, set the Data Points by adding them to the "main" environment.
    - The "main" environment can be thought of as your default set of data points. However, if your application needs more than one type of feedback, other environments can be created.

~~~~
override func viewDidLoad() {
    super.viewDidLoad()
        
    FeedbackEnvironment.add(with: [Emoji.bad, Emoji.ok, Emoji.good])
}
~~~~

<br><br>
## Presenting and Handing the Callbacks

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
## Creating Custom Data Points

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
