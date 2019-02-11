# MVVM Guidelines

* [Inputs - Outputs](#inputs---outputs)
  * [Principles](#principles)
  * [How-to](#how-to)
  * [Why Inputs - Outputs?](#why-inputs--outputs)
  * [Example](#example)
  * [Reference](#reference)

## Inputs - Outputs

#### Principles

inputs is a set of actions and events that have impacts on viewModel such as the tap action on a button, or the viewDidLoad event.
outputs represents changes that views should reflect.
Since ouputs may change over time, it’s best to return an Observable (in RxSwift context) for each ouput.
Behaviors defined in inputs should not be expressed as Variable because we don’t need the inputs to be obseravable.

#### How to

```swift
protocol LoginViewModelInputsType {
	var viewDidLoad: PublishRelay<Void> { get }
	var tapSubmit: PublishRelay<Void> { get }

}

protocol LoginViewModelOutputsType {
	var validInput: Driver<Bool> { get }
	var isLoading: Driver<Bool> { get }
	var loginSuccess: Driver<Void> { get }
	var loginFailure: Driver<ErrorMessage> { get }
}

protocol LoginViewModelType {
	var inputs: LoginViewModelInputsType { get }
	var ouputs: LoginViewModelOutputsType { get }
}
```

This is what LoginViewModel looks like:

```swift
final class LoginViewModel: LoginViewModelType, LoginViewModelInputsType, LoginViewModelOutputsType {
	var inputs: LoginViewModelInputsType { return self }
	var ouputs: LoginViewModelOutputsType { return self }

	// MARK: - Inputs
	let tapSubmit = PublishRelay<Void>()
	let email = PublishRelay<String>(value: "")

	...
}
```

#### Why Inputs / Outputs?

First of all, by using protocols like this, we achieve higher level of abstraction. Therefore, our code is more behavior-oriented and easier to test.

Another advantage of this protocol-based convention is readability in unit tests. Let’s take a look at the two simple tests below:

```swift
func test_When_PasswordIsEmpty_Then_InputIsInvalid() {
	viewModel.inputs.viewDidLoad()
	viewModel.inputs.type(email: "abc@xyz.com")

	// `grabLatestValue` is just an utility function we can write to retrieve
	// the latest value in the stream (during a specific period of time).
	// `RxBlocking` comes for the rescue.
	let validInput = grabLatestValue(viewModel.outputs.validInput, duration: 1)
	expect(validInput).to(beFalse())
}

func test_When_Submitting_Then_ShouldShowLoadingAndThenHideWhenCompleted() {
	viewModel.inputs.viewDidLoad()
	viewModel.inputs.type(email: "abc@xyz.com")
	viewModel.inputs.type(password: "Password0")
	viewModel.inputs.tapSubmit()

	let loadingStates = grabLatestValue(viewModel.outputs.isLoading, duration: 1)
	expect(loadingStates).to(beEqual([true, false]))
}
```

By looking at the codes related to inputs calls, we quickly have a sense of the scenarios we are trying to simulate. Similarly, what we expect to see are reflected upon outputs.

#### Reference

[1] https://github.com/kickstarter/native-docs/blob/master/vm-structure.md
[2] https://github.com/kickstarter/native-docs/blob/master/inputs-outputs.md
