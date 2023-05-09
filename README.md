# formik-to-react-hook-form-migration-guide

A guide for migrating projects to from [Formik](https://formik.org/) to [React Hook Form](https://react-hook-form.com/).

This guide will contain a lot of links and references to other libraries and documentation, but few code snippets of its
own. This is simply a maintenance choice, knowing that the JS ecosystem moves fast, and it is always best to reference
official documentation. This guide is intended to work best if viewed in conjunction with linked documentation,
examples, and of course your own application code.

## TL;DR

Migrating away from Formik appears to be a necessary eventuality. React Hook Form is an excellent alternative with
similar features and patterns, making migration less painful than you might think.

## Table of Contents
- [Why?](#why)
- [When?](#when)
- [How?](#how)
- [Strategy](#strategy)
- [Migration](#migration)
  - [Test Coverage](#test-coverage)
  - [Form Components](#form-components)
  - [Forms](#forms)
  - [Cleanup](#cleanup)
- [Migration Glossary](#migration-glossary)
- [Key Differences](#key-differences)
  - [Naming](#naming)
  - [Rendering](#rendering)
  - [Form validation](#form-validation)
  - [Submission](#submission)
  - [Validation Errors](#validation-errors)
  - [User Event Tests](#user-event-tests)
- [Disclaimer](#disclaimer)

## Why?

Formik is unmaintained with the last version at the time of writing published [over 2 years ago](https://www.npmjs.com/package/formik?activeTab=versions). 
It is only a matter of time before the project is more detrimental to include as a dependency than it is helpful.
There may even come a time when Formik is no longer compatible with modern versions of Node or React=, or other meta
frameworks.

React Hook Form has surpassed Formik in popularity based on [npm download statistics](https://www.npmjs.com/package/react-hook-form).
While the JS ecosystem is certainly volatile, and the future of any library will always involve some degree of
uncertainty, [React Hook Form is actively maintained](https://www.npmjs.com/package/react-hook-form?activeTab=versions).

If you weren't already convinced, [A maintainer of Formik has suggested not using Formik going forward, and instead
recommends React Hook Form](https://github.com/jaredpalmer/formik/discussions/3526).

## When?

_Now_ is the time to formulate your plan. 

_Now_ is also the best time to start the work.

Use this guide to estimate the effort and impact of the migration for your use case. Migration should start _before_ the
need is urgent. Swapping this library out while there is an urgent blocker, be it compatability or security concerns will
be a stressful, painful, and largely unrewarding experience.

## How?

This is what you're probably here for. Unsurprisingly, the answer is "It depends".

Your plan of attack should start by assessing two things:

1. _Where_ is Formik currently used?
2. _How_ is Formik currently used?

### 1. Where is Formik used? - Why does it matter?

If you have an application where Formik web forms are only a small portion of your application, congratulations, you're
going to have a pretty easy time with this migration! You could be looking at a scale of days or within a single sprint.

If your application is heavily form based, involves complex, dynamic, multi-step/multi-page forms... take a deep breath,
it's going to be fine, but will require a more nuanced approach to minimize conflicts and regressions.

### 2. How is Formik used? - Why does it matter?

There are two ways Formik is predominantly used, one of which will hopefully match what you see in your own application.

1. Using components like [`<Field />`](https://formik.org/docs/api/field) to wrap your form inputs.
2. Using custom components that implement hooks like [`useField()`](https://formik.org/docs/api/useField).

You may even see _both_ in your application, especially if it was developed over long periods of time and/or by 
different folks. One is not necessarily more difficult to migrate than the other, as React Hook Form offers 
near-equivalents for both cases. Knowing which documents to start with is half the battle, so let's get you pointed in
the right direction right now.

- If your forms use `<Field />`, you're going to [`register`](https://react-hook-form.com/api/useform/register/) them with
React Hook Form's top-level [`useForm()`](https://react-hook-form.com/api/useform/) hook.
- If your form components implement `useField()`, you're going to use [`useController()`](https://react-hook-form.com/api/usecontroller/)
instead.

> ℹ️ A full list of equivalent utilities, can be found in the [glossary](#migration-glossary) below.

Of course when implementing a new library it's also a good idea to get a feel for all the functionality available by 
scanning the rest of the documentation as well. React Hook Form's [Get Started](https://react-hook-form.com/get-started/)
documentation is a great entrypoint.


## Strategy

By now you've taken stock of how much code is impacted, and you have a general idea of where to start based on how you
use Formik. What may not be obvious is how you should start, and what the migration looks like. Is it one PR? A 
long-lived branch accompanying regular development? An epic-sized effort spanning several sprints and multiple PRs?

All great questions. And the answer is, again, "It depends".

This guide isn't going to make recommendations on branching strategy, to not conflate the two, but its worth
mentioning that your team's branching and delivery strategy may impact your approach. Since Truss prefers incremental
delivery on Agile teams, this guide will favor approaches that work under that framework.

In general, it is best to migrate, test, release, and test some more the smallest piece of your application's form 
infrastructure first. Breaking down the migration into manageable chunks is key.

Keep the following in mind while breaking down the effort:
- Formik and React Hook Form _can_ coexist within the same application, but it is _not_ recommended (and my not be 
possible) to use both within the same form, depending on your implementation.
- Form components can only be effectively controlled by one form library. It is _not_ recommended to attempt to make 
contort your existing form components so that they can be used within the context of Formik or React Hook Form.
- The test coverage of individual form components, as well as the forms themselves, and the end-to-end flow involving
affected forms will directly and proportionately increase your confidence in the changes being made.
- Tests that are written for Formik-based pieces of the applications should be agnostic of the form library choice, and
therefore should not need to change outside of setup (context providing components and mocking, potentially) in unit
tests.

Given the context and constraints provided up until this point, and a little experimenting of our own, the ideal chunks
we recommend evaluating are
- Increasing test coverage to 100% (or as close as is pragmatic, knowing each untested path or permutation 
increases the chance of leaking regressions), of component unit tests, and end-to-end or integration tests that exercise 
the form portions of your app.
- Build React Hook Form components equivalents to your current Formik-aware form components.
  - Applying existing component tests to the new components is a great way to get an early feel for compatability, 
  before making user-facing changes.
- Replace each Formik form (or form page in a multipage form) with a React Hook Form equivalent.
  - Start small, simple, and low risk to build experience with React Hook Form while chipping away. Build confidence
  with React Hook Form while learning which pieces are trickier or more time-consuming to better inform the refinement
  of the rest of the migration effort

## Migration

By now you should have your Agile mise en place at the ready. You've evaluated your circumstances and broken down at
least a few chunks of work, even if just exploratory. What that looks like will be completely up to you, your team's
process and prioritization, and the forms that need to be rebuilt. Hopefully you've taken the advice in the earlier
sections of this guide and focused prioritization and refinement on the smallest, least complex, and lowest-risk form
first.

### Test Coverage

The rest of this guide assumes you've amped up test coverage wherever it might have been lacking, so we won't go into
detail here. Do it if you need it, you'll thank yourself later.

### Form components

First thing's first, you'll need all the right building blocks to replace your first Formik form. Remember the "[how](#2-how-is-formik-used---why-does-it-matter)"
we talked about before? This is a step you can skip if your form inlines all the inputs with the Formik `<Field />`
wrapper. 

Since many Truss projects use [ReactUSWDS](https://github.com/trussworks/react-uswds) with Formik, we often create 
reusable custom components that implement Formik's `useField()` hook. The rest of this section will detail how to create 
a React Hook Form equivalent that you can use to rebuild your form. If you want to skip ahead and see what such a
component built with React Hook Form might look like, check out [this one](https://github.com/trussworks/unemployment-insurance-modernization-demo/blob/main/src/components/form/fields/YesNoQuestion/YesNoQuestion.tsx)
and the [corresponding unit tests](https://github.com/trussworks/unemployment-insurance-modernization-demo/blob/main/src/components/form/fields/YesNoQuestion/YesNoQuestion.test.tsx)
in our [Unemployment Insurance Modernization Demo](https://github.com/trussworks/unemployment-insurance-modernization-demo/)
repository.

Since the form library that these components use is probably an implementation detail, start your component migration
with the following goal in mind:

> _I should be able to update my form component's import to the React Hook Form version instead of the Formik version in 
> a React Hook Form controlled form, and nothing else about my JSX/TSX markup related to this component should need to
> change._

If you can stick to that goal, your existing unit tests will largely be transferable, and the migration of parent
components that implement your form component will be straightforward:

1. Create a copy of your custom Formik-aware component 
   - Follow your project's naming and organizational conventions
   - Know that the Formik-aware should be removed when the entire migration effort is completed (see [Cleanup](#cleanup) 
   for more details)
2. Use the [Migration Glossary](#migration-glossary) to evaluate which Formik utilities will need to be replaced, and
what to replace them with
3. Whether you choose to take a TDD approach or not, setting up your unit tests to run against the React Hook Form 
version of your component is the fastest way to create a tight feedback loop on your progress. 
   - It is recommended to do this as soon as you are able to. 
   - Understandably, especially if context providers and mocking are involved, and you are new to React Hook Form, you
   may not be able to do this confidently
   - If you use [Storybook](https://storybook.js.org/), it can also be a great way to manually sanity test your
   component with a reasonable feedback loop
4. Replace Formik hooks and utilities with the React Hook Form equivalents, extract the necessary return props, and 
ensure they are properly wired into your resulting markup 
5. Once all Formik references have been removed, iterate on your tests and the component as necessary until all tests
pass

### Forms

At this point, you should have all the building blocks ready to rebuild your form. It's time to implement them.

If you want to skip ahead to what a complete React Hook Form implementation with custom components, schema validation,
and dynamic, conditional rendering looks like, check out [this example](https://github.com/trussworks/unemployment-insurance-modernization-demo/blob/main/src/pages/identity/identity.tsx) 
from our [Unemployment Insurance Modernization Demo](https://github.com/trussworks/unemployment-insurance-modernization-demo/)
repository.

Similar to the [Component Migration](#form-components) section, let's set a goal:

> _I should be able to update my form implementation from Formik to React Hook Form without perceptible differences to
> users, or interaction based tests._

1. No matter how your form is built, you'll need to start with React Hook Form's [`useForm()`](https://react-hook-form.com/api/useform/)
hook. At this point you'll set up your initial/default values, [schema validation](#form-validation), default behaviors,
and [submit handlers](#submission).
2. Register your fields with React Hook Form
   - If your form uses inlined inputs with the Formik `<Field />`, use the `useForm()` [`register`](https://react-hook-form.com/api/useform/register/)
   utility instead. Depending on the complexity of your form, that may be all you need to do!
   - If your form uses custom components, update your imports to point to the React Hook Form versions you created in 
   the previous section. 
3. If your form needs to be aware of realtime user entry, rather than on submit (e.g. conditionally hiding, showing, 
setting, and/or clearing fields) set up watches with the [`useWatch()`](https://react-hook-form.com/api/usewatch/) hook.
4. Start relying on your feedback loops by updating and running your unit tests
   - If you use Storybook, this is also a good time to start manual smoke testing. 
   - You may also want to try running the app at this point and begin manual smoke testing
5. Continue updating your form and corresponding tests if needed (probably predominantly context providers or mocking),
until all unit tests pass
6. Ensure your relevant user flows still work with integration and/or end-to-end tests, and manual testing

### Cleanup

1. Remove your dependency to Formik 🚮
2. Remove any references to (now hopefully unused) Formik-aware code
3. Release and deploy a new version of your project free of Formik at last

🎉 You did it! Cheers to you! 🎉

## Migration Glossary

Below is a list of equivalent or near-equivalent utilities. Did you encounter one that's missing? Please open a PR to 
this guide and add it!

| Formik                                                               | React Hook Form                                                                                                                                                                                                 | Additional details                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|----------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`<Formik />`](https://formik.org/docs/api/formik)                   | [`useForm()`](https://react-hook-form.com/api/useform/)                                                                                                                                                         | If you were using `<Formik />` as a context provider in your tests, consider [`<FormProvider />`](https://react-hook-form.com/api/formprovider/) to do the same with React Hook Form                                                                                                                                                                                                                                                                 |
| [`useFormik()`](https://formik.org/docs/api/useFormik)               | [`useForm()`](https://react-hook-form.com/api/useform/)                                                                                                                                                         |                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| [`withFormik()`](https://formik.org/docs/api/withFormik)             | [`useForm()`](https://react-hook-form.com/api/useform/)                                                                                                                                                         |                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| [`<Field />`](https://formik.org/docs/api/field)                     | `useForm()`'s [`register`](https://react-hook-form.com/api/useform/register/) utility                                                                                                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| [`useField()`](https://formik.org/docs/api/useField)                 | [`useController()`](https://react-hook-form.com/api/usecontroller/)                                                                                                                                             |                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| [`useFormikContext()`](https://formik.org/docs/api/useFormikContext) | [`useFormContext()`](https://react-hook-form.com/api/useformcontext/) <br /> [`useFormState()`](https://react-hook-form.com/api/useformstate/) <br /> [`useWatch()`](https://react-hook-form.com/api/usewatch/) | `useFormContext()` provides access to everything that your top-level `useForm()` hook provides. This is where most of your utilties are for imperatively interacting with the form. <br /> `useFormState()` provides meta information about the form's current state (like touched state, errors, submitCount, validation status, etc. <br /> `useWatch()` enables realtime access to a field's value (see [rendering](#rendering) for more details) |

There are a lot of hooks, components, utilities, terms, and techniques in each of these libraries. React Hook Form can get
much more advanced than this guide covers. Make sure to check out official documentation for [TypeScript support](https://react-hook-form.com/ts/)
if you use it, and [advanced techniques](https://react-hook-form.com/advanced-usage/) if your use case isn't covered in
this guide.

## Key Differences

### Naming

React Hook Form gets a little more specific about naming than Formik's generic, self-described "Formik bag", some
subset of which is provided by various Formik utilities. With React Hook Form we found there were more incidents of
naming conflicts with properties and functions like `name` and `onChange`. When aliasing the React Hook Form provided
properties and functions we found it useful to prefix with `hookForm` (e.g. `hookFormName` or `hookFormOnChange`). 

If you aren't already aware, "React Hook Form" is a bit of a mouthful, and also a keyboardful (which my fingers are 
currently keenly aware of as this is the _only_ section of this guide where we don't spell out the entire name).
Shortening references to "Hook Form" or "RHF" is recommended. Pick a convention and go with it, no one will blame you.

The two libraries use similar, but not identical, terminology to reference the same concepts (e.g. Formik's 
[initialValues](https://formik.org/docs/api/formik#initialvalues-values) vs React Hook Form's [defaultValues](https://react-hook-form.com/api/useform/#defaultValues))

### Rendering

Where Formik renders _liberally_ by default, React Hook Form is optimized for fewer re-renders. When it is important to
know the realtime value of a React Hook Form managed field, React Hook Form's [`useWatch()`](https://react-hook-form.com/api/usewatch/)
will enable that for you.

### Form validation

React Hook Form supports [native validations](https://react-hook-form.com/get-started/#Applyvalidation) for only the
simplest (but likely most common) validation rules.

If you're reading this guide, your project likely uses Formik in combination with a validation library like Zod or Yup.
Fortunately, React Hook Form also supports [schema-based validation](https://react-hook-form.com/get-started/#SchemaValidation)
with popular libraries. If you're already using one of the supported validation libraries, simply plug in your existing
schema.

### Submission

Where Formik provides [insight into the magic encapsulated in its form submission process](https://formik.org/docs/guides/form-submission),
React Hook Form is [a little less explicit in this area](https://react-hook-form.com/api/useform/handlesubmit/).

Formik uses a single submit handler callback during submission. React Hook Form separates the SubmitHandler (the 
callback to run on submit with no validation errors) from a SubmitErrorHandler (the callback to run on submit when 
validation errors occur).

The structure of the submitted values object is fortunately identical to that of Formik, so everything downstream of
your form should have no reason to change.

### Validation Errors

One of the nicest features React Hook Form offers is a ref to each input in error (and a default behavior of focusing 
the first one), allowing vastly simplified programmatic navigation to fields in error, particularly useful 
for longer, single-page forms.

> ℹ️ The "first" error is not necessarily the first to appear in the DOM, as React Hook Form has no awareness of render
order. Custom logic may be required (as it would be with Formik) to find the element with the highest position, if 
desired.

### User Event Tests

The Javascript ecosystem moves fast, much faster than this guide will be updated. Hopefully by the time you are reading
this, this is no longer an issue.

At the time of writing, note that interacting with React Hook Form fields using React Testing Library and User Events
will likely cause you to see a greater incidence of the dreaded `Warning: An update to YourComponent inside a test was 
not wrapped in act(...)`.

This is the one area that has been trickier to resolve than with Formik, and ultimately unsatisfactory. For now know
this: You will have to break a testing library best practice in order to silence some of these warnings, and it will
feel bad to do so.

[This discussion](https://github.com/orgs/react-hook-form/discussions/4232) will have more up-to-date information
than this migration guide, please look there for answers. [This comment in particular](https://github.com/orgs/react-hook-form/discussions/4232#discussioncomment-3498800) 
is a sign that it might not be you or your tests that are problematic. You might need that redundant `act` block after
all.

## Disclaimer

If React Hook Form has changed enough that this guide is no longer a helpful reference, it's more than likely far beyond
prudent timing to have completed your migration away from Formik. 

To any readers this statement applies to: Hello, and condolences.