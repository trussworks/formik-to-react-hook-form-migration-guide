# formik-to-react-hook-form-migration-guide

A guide for migrating projects to from [Formik](https://formik.org/) to [React Hook Form](https://react-hook-form.com/).

## TL;DR

Migrating away from Formik appears to be a necessary eventuality. React Hook Form is an excellent alternative with
similar features and patterns, making migration less painful than you might think.

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

Of course when implementing a new library it's also a good idea to get a feel for all the functionality available by 
scanning the rest of the documentation as well. React Hook Form's [Get Started](https://react-hook-form.com/get-started/)
documentation is a great entrypoint.

> ℹ️ TODO A full list of equivalent utilities, can be found [below](#TODO-link-to-section-header)

### Migration Strategy

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
  - Start small and low risk to build experience with React Hook Form while chipping away.

## Migration

