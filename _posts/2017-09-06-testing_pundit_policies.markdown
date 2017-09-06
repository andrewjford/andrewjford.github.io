---
layout: post
title:  "Testing Pundit Policies"
date:   2017-09-06 18:59:28 +0000
---


I have been working my way through [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec) by Aaron Sumner. It has been a great resource for learning common testing approaches with RSpec. I have been building out my tests for a [previous project](https://andrewjford.github.io/2017/07/04/auditrequest/) while going through the book.

When I came to the controller testing section I had to take a detour into testing Pundit. The app I was building tests for uses Pundit for authorization. I chose Pundit because it allowed me to have a simple, structured way to control user access.

## Pundit Overview

Pundit authorization centers around a policy class that relates to each model; the policy class sets accessibility to that model. In my application I have projects that can only be created by admin or manager level users. Users roles are: admin, manager, auditor or client (in descending levels of authorization).

```
class ProjectPolicy < ApplicationPolicy
  def create?
    user.admin? || user.manager?
  end
```

Above is a portion of the Project policy class. It is common for the user roles to be created as enums to make them more readable. The above code allows only admins and managers to access the create action.

```
// projects_controller
def create
    @project = Project.new(project_params)
    authorize @project

    # rest of create code....
  end
```

In the create action of the projects controller we check whether the current user has access with `authorize @project`. Pundit checks the user against the Project Policy create? and grants or denies based on the boolean evaluation in the policy. If a policy denies access a standard authorization error message is shown and the user is redirected to the homepage.

## Testing Pundit

There are two common ways of testing Pundit with RSpec. One is described [here](https://www.thunderboltlabs.com/blog/2013/03/27/testing-pundit-policies-with-rspec/), using custom matchers. The second method is to use the mini-DSL, which is what I chose to do.

To use the DSL I first added `require "pundit/rspec"` to the top of the `spec_helper.rb` file. I then created a new `spec/policies` folder for my Pundit policy tests.

To perform basic testing of the policies we set a subject which is the policy being tested, in this example the ProjectPolicy. We then specify what permissions of the policy to test and create a block for that. Here we are testing the new and create actions of the projects controller.

```
require 'spec_helper'

describe ProjectPolicy do
  subject { ProjectPolicy }

  permissions :new?, :create? do
    it "denies access if a client" do
      expect(subject).not_to permit(User.new(role: 3), Project.new())
    end
    # other code...

    it "grants access if user is a manager" do
      expect(subject).to permit(User.new(role: 1), Project.new())
    end
  end
```

We then test whether we expect a given user to be permitted or not according to this Pundit policy. Role 3 is for client level users who should not have access. Role 1 corresponds to a manager, which we expect to be permitted.

The show action for projects has different restrictions than create and new. It only allows users that have been assigned to the project (or admins).

```
def show?
  user.admin? || record.users.include?(user)
end
```

We test that non admins by default cannot access a project. We then create a user and test that that user is granted access when they are included in the project.

```
permissions :show? do
    it "denies access to manager if not assigned to project" do
      expect(subject).not_to permit(User.new(role: 1), Project.new())
    end
    # other code...

    it "grants access if user is assigned" do
      @user = FactoryGirl.create(:user)

      expect(subject).to permit(@user,
        Project.create(title: "Test1", user_ids: [@user.id]))
    end
  end
```

Hopefully this post was helpful to those looking at testing Pundit policies in RSpec. I was unable to find many resources on using this mini-DSL from `pundit/rspec`, but it seems like an efficient option to test Pundit policies.

