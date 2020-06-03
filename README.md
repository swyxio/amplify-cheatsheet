# amplify-cheatsheet
amplify stuff i use all the time

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [AppsSync @auth directive](#appssync-auth-directive)
  - [Public Read + Private Create, Update, Delete](#public-read--private-create-update-delete)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## AppsSync @auth directive

```graphql
# When applied to a type, augments the application with
# owner and group-based authorization rules.
directive @auth(rules: [AuthRule!]!) on OBJECT, FIELD_DEFINITION
input AuthRule {
  allow: AuthStrategy!
  provider: AuthProvider
  ownerField: String # defaults to "owner" when using owner auth
  identityClaim: String # defaults to "username" when using owner auth
  groupClaim: String # defaults to "cognito:groups" when using Group auth
  groups: [String]  # Required when using Static Group auth
  groupsField: String # defaults to "groups" when using Dynamic Group auth
  operations: [ModelOperation] # Required for finer control

  # The following arguments are deprecated. It is encouraged to use the 'operations' argument.
  queries: [ModelQuery]
  mutations: [ModelMutation]
}
enum AuthStrategy { owner groups private public }
enum AuthProvider { apiKey iam oidc userPools }
enum ModelOperation { create update delete read }

# The following objects are deprecated. It is encouraged to use ModelOperations.
enum ModelQuery { get list }
enum ModelMutation { create update delete }
```

### Public Read + Private Create, Update, Delete

```graphql
  @auth(
    rules: [
      # allow all authenticated users ability to create posts
      # allow owners ability to update and delete their posts
      # allow all authenticated users to read posts
      { allow: owner, operations: [create, update, delete] }
      # allow all guest users (not authenticated) to read posts
      { allow: public, operations: [read] }
    ]
  )
```

But `public` needs API key, `owner` needs User Pools. So you need to setup both default and secondary auth with BOTH cognito user pools and API key:

```bash
# amplify add auth # make sure you have auth
amplify update api
? Please select from one of the below mentioned services: GraphQL
? Select from the options below Update auth settings
? Choose the default authorization type for the API Amazon Cognito User Pool
Use a Cognito user pool configured as a part of this project.
? Configure additional auth types? Yes
? Choose the additional authorization types you want to configure for the API API key
API key configuration
? Enter a description for the API key: babys first api key
? After how many days from now the API key should expire (1-365): 7
```

but then how to access it??? hmm.

- https://www.youtube.com/watch?v=WM9WcEHxJmg
