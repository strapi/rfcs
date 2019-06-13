# Strapi RFCs

Some important changes in Strapi require some thoughts to be put into the design phase before starting workin on a PR.

The RFC (Request For Comments) process will help us create consensus among the core team and include as much feedback as possible from the community.

## Process

- Clone the repository.
- Copy the `xxxx-template.md` (leave the xxxx at the beginning) file to the rfcs folder.
- Write your RFC in the copied file
- Make a Pull request (create a link to the rendered RFC file in the PR description)
- Wait for comments and try building consensus while integrating feedbacks.
- Eventually the core team will decide whether the RFC should be accepted or rejected.

## Final Step

- In case of rejection a core team member will close the PR with a comment explaining the rational for rejection.
- In case the proposal is accepted the PR will be merge and the RFC will become 'active'

## Active RFCs

Once an RFC becomes active, then authors may implement it and submit the feature as a pull request to the Strapi repo.
Becoming 'active' does not mean the feature will ultimately be merged; it does mean that the core team has agreed to it in principle and are amenable to merging it.

Furthermore, the fact that a given RFC has been accepted and is 'active' implies nothing about what priority is assigned to its implementation, nor whether anybody is currently working on it.

## Reviewing RFCs
Each week the team will attempt to review some set of open RFC pull requests.

**Strapi's RFC process owes its inspiration to the [React RFC process], [Yarn RFC process], [Rust RFC process], and [Ember RFC process]**

[React RFC process]: https://github.com/reactjs/rfcs
[Yarn RFC process]: https://github.com/yarnpkg/rfcs
[Rust RFC process]: https://github.com/rust-lang/rfcs
[Ember RFC process]: https://github.com/emberjs/rfcs