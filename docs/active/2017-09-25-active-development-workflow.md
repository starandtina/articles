# 活跃网络前端开发流程

1. Assign tickets
>Team members should be proactive to pick up the story and assign the corrsponing story to themselves, or the team lead will assign the stories to the team members

1. Pull the latest code base based on master branch
>The owner of story **must** goes through the requiements and investivgates the story as soon as possible beforehand, making sure that you know what we need to build. Find the BA/QA if needed and should add your sub-tasks to the story
If necessary,

1. Coding based on your feature branches which is named as yourDomainAccount-ticket-concise-description, for example, **bxong-ANE-41116-pagination-update**

1. Once the story is assigned, the owner of story should mark the **Assignee** as yourself

1. If the SP is more than **13** or the story would touch our core code base(such as middlewares) or shared components, the owner should create the detailed design document
    - Think more before do it then you will find it will cost less time on implementation
    - Once the design document is done, the ownere should schedule a design meeting and review the design document with the stakeholders together
    - Once the design review is passed, the owner could start the coding according to the design
    - If anything related with the design has changed, inform your team members as soon as you can

1. Dev depends on your time and deliverables, as well as considering more tiem for writing lots of tests

1. Test on your local box and make sure it's fully tested

1. Log your time and mark sub tasks(JIRA, GitLab issues) as closed after the code is completed

1. Commit code changes based on your feature branch

    >In terms of the Git commit message format, please refer to [Git Commit Guidelines](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#-git-commit-guidelines)

1. Create **Merge Request** for Code Review
    - Add appropriate title, description, related JIRA ticket link, etc.
    - Add labels, such as **Code Review**, **Feature**, **Enhancement**, **Bug** or **Test**
    - Mark **Assignee** as your coding parter, for example, Chris will add Ashley as the **Assignee** because they all work on the arc-ui project togther
    - Mention **Khalil Zhang** or other team members(if you think it's necessary) in the comment then requesting code review from him
    - Review fewer than 200-400 lines of code at a time, avoiding submitting tens of files or thousands of linkes of code :(
    - Every Merge Request would trigger the CI pipeline which would run your tests and generate coverage reports on every build. If the minimum threshold enforcement for coverage resutls(90%) is not met, the unit testing framework(such as Jest) should return failture then the build will fail and your Merge Request will not be merged. It's only act on new code.

    >The point of our code review is to eliminate as many defects as possible, regardless of who caused the error. Also, we should strictly follow **pre-commit** code review.
While working on a new feature, Bill(for example) will cut a branch from the current version of our master and work exclusively on that branch, I'm sure most of you are familar with this approach. However, before Bill could reintegrate the changes into master, Shirly, Khalil or another qualified engineer **must** review his work and give him the stamp of approval: **LGTM**(looks good to me)
If Shirly or Khalil has an issue or problem with the way Bill has desinged or coded for the work, they should have a discussion until they reach an agreement. Or if Shirly has no issues or problems, she could LGTM the work right way.

1. Once the Code Review is done, accept merge request and remove the **Code Review** label

    >Most of time, you would see the merge conflicts, please resolve these merge conflicts beforehand. Recommend using `git rebase` to get a cleaner history.
    >Please make sure **Remove source branch when merge request is accepted** and **Squash commits when merge request is accepted** checkboxes are always checked whenever creating the MR. Because the source branch would be deleted once the MR is accepted, at the same time it would tidy up the
commit history of a branch when accepting a merge request. It applies all of the changes in the merge request as a single commit,
and then merges that commit using the merge method set for the project.

1. Mark the respective JIRA tasks as completed or done, then re-assgin the store to QA for testing.

    >You'd better add some helpful comments for testing, such as the project version, testing scope, unfinished tasks, etc.

## Important notes

* Please make sure the master build is successful after your changes are merged
* Please make sure you test your changes in integration environments(or local environement if not) after your changes are merged and update the MR
* It’s a basic practice within software development to keep things that will have the same reason to change together and separate the things that have other reasons to change (Single Responsibility Principle). This same principle is applicable to MRs, thus **it’s a good practice to limit them to a single feature (or reason to change)**:

    * It’s easy to review changes as their scope is concise and constrained
    * It’s simpler to revert changes (as related changes would be within the same MR)

Please keep it in mind and stick to it, otherwise maybe you need to pay the Pepsi.

## Code Convention

### Blank lines

In general, code should look like a series of paragraphs rather than one continuous blob of text. Blank lines should be used to separate related lines of code from unrelated lines of code. Blank lines should be used to separate related lines of code from unrelated lines of code. The goal is to make our code easy to read.

In general, it’s a good idea to also add blank lines:

- Between methods
- Between the local variables in a method and its first statement
- Before a multiline or single-line comment
- Between logical sections inside a method to improve readability

