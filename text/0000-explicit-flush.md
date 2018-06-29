- Feature Name: explicit_flush
- Start Date: 2018-06-29
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: `#[explicit_flush]` annotation to mark functions to be explicitly called before destructor

Object functions can be annotated with `#[explicit_flush]`. When a function is annotated,
user is advisable to explicitly call than function prior to destroying the object.

# Motivation
[motivation]: #motivation

There's no reliable way to handle errors in destructors.

Suppose you have a buffered writer object. User called `write` function,
some data is buffered inside the object, and this object is not flushed.

Then object is destroyed. Most users either:
* forgot to explicitly call `flush`
* simply assume that data will be properly flushed in destructor

Here comes a problem, what to do in destructor:

* if destructor simply release resources, but doesn't flush data, it leads to bugs
  (e. g. sometimes data remain unwritten, and these bugs are hard to spot)
* if destructor flushes data, but ignores error, there's a chance of data loss
  (user assumes data is written, but it is not written, because)


# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Explain the proposal as if it was already included in the language and you were teaching it to another Rust programmer. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- Explaining how Rust programmers should *think* about the feature, and how it should impact the way they use Rust. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.
- If applicable, describe the differences between teaching this to existing Rust programmers and new Rust programmers.

For implementation-oriented RFCs (e.g. for compiler internals), this section should focus on how compiler contributors should think about the change, and give examples of its concrete impact. For policy RFCs, this section should provide an example-driven introduction to the policy, and explain its impact in concrete terms.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

Language gets another method-level annotation `#[explicit_flush]`.

Rust language gets built-in lint which fires if a normal program flow leading to destructor
doesn't have a last call to a flushing functions.

Example

```
{
    let my_write = ...
    my_write.write("hello");
    my_write.flush()
}
```

TODO: explain ? return.

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

* Yet another complication of the language.
* Write failures are practically rare, for the most use cases it's enough
  to simply terminate or ignore error in destructor, and data-sensitive program developers
  (like databases) developers can just remember to flush data explicitly.

# Rationale and alternatives
[alternatives]: #alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?

# Prior art
[prior-art]: #prior-art

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- For language, library, cargo, tools, and compiler proposals: Does this feature exist in other programming languages and what experience have their community had?
- For community proposals: Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the lessons from other languages, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other languages.

Note that while precedent set by other languages is some motivation, it does not on its own motivate an RFC.
Please also take into consideration that rust sometimes intentionally diverges from common language features.

# Unresolved questions
[unresolved]: #unresolved-questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?
