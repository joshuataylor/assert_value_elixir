# assert_value

Checks that two values are same and "magically" replaces expected value
with the actual in case the new behavior (and new actual value) is correct.

For now supports two kind of expected arguments: string heredocs or files

# Screencast

![Screencast](https://github.com/assert-value/assert_value_screencasts/raw/master/elixir/screencast.gif)


## Requirements

Elixir ~> 1.4

## Installation

Add AssertValue as a test env dependency to your Mix project

```elixir
defp deps do
  [{:assert_value, git: "git@github.com:assert-value/assert_value_elixir.git", only: :test}]
end
```
Add to config/test.exs to avoid timeouts

```elixir
# Avoid timeouts while waiting for user input in assert_value
config :ex_unit, timeout: :infinity
config :my_app, MyApp.Repo,
  timeout: :infinity,
  ownership_timeout: :infinity
```

## Usage

### Testing String Values

It is better to start with no expected value

```elixir
import AssertValue

test "fresh start" do
  assert_value "foo"
end
```
Then run your tests as usual with "mix test".
As a result you will see diff between expected and actual values:
```
<Failed Assertion Message>
    "foo"

-
+foo

Accept new value [y/n]?
```
If you accept the new value your test will be automatically modified to
```elixir
test "fresh start" do
  assert_value "foo" == """
  foo
  """
end
```

### Testing Values Stored in Files

Sometimes test string is too large to be inlined into the test source.
Put it into the file instead.

```elixir
assert_value "foo" == File.read!("test/log/reference.txt")
```
AssertValue is smart enough to recognize File.read! and will update file contents
instead of test source. If file does not exists it will be created and no error
will be raised despite default File.read! behaviour.

### Running Tests Non-interactively

AssertValue will autodetect whether it is running interactively (in a
terminal), or non-interactively (e.g. continuous integration).
When running interactively it will ask about each diff. When running
non-interactively it will reject all diffs by default, and will work
like default ExUnit's assert.

To override autodetection use ASSERT_VALUE_ACCEPT_DIFFS environment variable
with one of three values: "ask", "y", "n"
```
# Ask about each diff. Useful to force interactive behavior despite
# autodetection.
ASSERT_VALUE_ACCEPT_DIFFS=ask mix test

# Reject all diffs. Useful to force default non-interactive mode when running
# in an interactive terminal.
ASSERT_VALUE_ACCEPT_DIFFS=n mix test

# Accept all diffs. Useful to update all expected values after a refactoring.
ASSERT_VALUE_ACCEPT_DIFFS=y mix test
```

## Notes and Known Issues

  * AssertValue requires left argument to be a string. However it will accept
    everything with implemented String.Chars protocol (Atom, BitString, Char List,
    Integer, Float). It will raise AssertValue.ArgumentError in all other cases.
  * Right argument should be in the form of string heredoc starting and ending
    with """ or File.read!.
  * Using plain strings as right argument will cause incorrect test source changes.
  * Using other types as right argument will raise AssertValue.ArgumentError.

## License

This software is licensed under [the MIT license](LICENSE).
