# Casbin.NET

**News**: still worry about how to write the correct Casbin policy? `Casbin online editor` is coming to help! Try it at: http://casbin.org/editor/

## Get Started

```
dotnet add package NetCasbin --version 1.0.0
```

## Table of contents

- [Supported models](#supported-models)
- [How it works?](#how-it-works)
- [Features](#features)
- [Installation](#installation)
- [Documentation](#documentation)
- [Online editor](#online-editor)
- [Tutorials](#tutorials)
- [Get started](#get-started)
- [Policy management](#policy-management)
- [Policy persistence](#policy-persistence)
- [Policy consistence between multiple nodes](#policy-consistence-between-multiple-nodes)
- [Role manager](#role-manager)
- [Benchmarks](#benchmarks)
- [Examples](#examples)
- [Middlewares](#middlewares)
- [Our adopters](#our-adopters)

## Supported models

1. [**ACL (Access Control List)**](https://en.wikipedia.org/wiki/Access_control_list)
2. **ACL with [superuser](https://en.wikipedia.org/wiki/Superuser)**
3. **ACL without users**: especially useful for systems that don't have authentication or user log-ins.
4. **ACL without resources**: some scenarios may target for a type of resources instead of an individual resource by using permissions like `write-article`, `read-log`. It doesn't control the access to a specific article or log.
5. **[RBAC (Role-Based Access Control)](https://en.wikipedia.org/wiki/Role-based_access_control)**
6. **RBAC with resource roles**: both users and resources can have roles (or groups) at the same time.
7. **RBAC with domains/tenants**: users can have different role sets for different domains/tenants.
8. **[ABAC (Attribute-Based Access Control)](https://en.wikipedia.org/wiki/Attribute-Based_Access_Control)**: syntax sugar like `resource.Owner` can be used to get the attribute for a resource.
9. **[RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer)**: supports paths like `/res/*`, `/res/:id` and HTTP methods like `GET`, `POST`, `PUT`, `DELETE`.
10. **Deny-override**: both allow and deny authorizations are supported, deny overrides the allow.
11. **Priority**: the policy rules can be prioritized like firewall rules.

## How it works?

In Casbin, an access control model is abstracted into a CONF file based on the **PERM metamodel (Policy, Effect, Request, Matchers)**. So switching or upgrading the authorization mechanism for a project is just as simple as modifying a configuration. You can customize your own access control model by combining the available models. For example, you can get RBAC roles and ABAC attributes together inside one model and share one set of policy rules.

The most basic and simplest model in Casbin is ACL. ACL's model CONF is:

```ini
# Request definition
[request_definition]
r = sub, obj, act

# Policy definition
[policy_definition]
p = sub, obj, act

# Policy effect
[policy_effect]
e = some(where (p.eft == allow))

# Matchers
[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act

```

An example policy for ACL model is like:

```
p, alice, data1, read
p, bob, data2, write
```

It means:

- alice can read data1
- bob can write data2

We also support multi-line mode by appending '\\' in the end:

```ini
# Matchers
[matchers]
m = r.sub == p.sub && r.obj == p.obj \
  && r.act == p.act
```

Further more, if you are using ABAC, you can try operator `in` like following in Casbin **golang** edition (jCasbin and Node-Casbin are not supported yet):

```ini
# Matchers
[matchers]
m = r.obj == p.obj && r.act == p.act || r.obj in ('data2', 'data3')
```

But you **SHOULD** make sure that the length of the array is **MORE** than **1**, otherwise there will cause it to panic.

For more operators, you may take a look at [govaluate](https://github.com/Knetic/govaluate)

## Features

What Casbin does:

1. enforce the policy in the classic `{subject, object, action}` form or a customized form as you defined, both allow and deny authorizations are supported.
2. handle the storage of the access control model and its policy.
3. manage the role-user mappings and role-role mappings (aka role hierarchy in RBAC).
4. support built-in superuser like `root` or `administrator`. A superuser can do anything without explict permissions.
5. multiple built-in operators to support the rule matching. For example, `keyMatch` can map a resource key `/foo/bar` to the pattern `/foo*`.

What Casbin does NOT do:

1. authentication (aka verify `username` and `password` when a user logs in)
2. manage the list of users or roles. I believe it's more convenient for the project itself to manage these entities. Users usually have their passwords, and Casbin is not designed as a password container. However, Casbin stores the user-role mapping for the RBAC scenario.

## Installation

```
go get github.com/casbin/casbin
```

## Documentation

https://casbin.org/docs/en/overview

## Online editor

You can also use the online editor (http://casbin.org/editor/) to write your Casbin model and policy in your web browser. It provides functionality such as `syntax highlighting` and `code completion`, just like an IDE for a programming language.

## Tutorials

https://casbin.org/docs/en/tutorials

## Get started

1. New a Casbin enforcer with a model file and a policy file:

   ```go
   e := casbin.NewEnforcer("path/to/model.conf", "path/to/policy.csv")
   ```

Note: you can also initialize an enforcer with policy in DB instead of file, see [Persistence](#persistence) section for details.

2. Add an enforcement hook into your code right before the access happens:

   ```go
   sub := "alice" // the user that wants to access a resource.
   obj := "data1" // the resource that is going to be accessed.
   act := "read" // the operation that the user performs on the resource.

   if e.Enforce(sub, obj, act) == true {
       // permit alice to read data1
   } else {
       // deny the request, show an error
   }
   ```

3. Besides the static policy file, Casbin also provides API for permission management at run-time. For example, You can get all the roles assigned to a user as below:

   ```go
   roles := e.GetRoles("alice")
   ```

See [Policy management APIs](#policy-management) for more usage.

4. Please refer to the `_test.go` files for more usage.

## Policy management

Casbin provides two sets of APIs to manage permissions:

- [Management API](https://github.com/casbin/casbin/blob/master/management_api.go): the primitive API that provides full support for Casbin policy management. See [here](https://github.com/casbin/casbin/blob/master/management_api_test.go) for examples.
- [RBAC API](https://github.com/casbin/casbin/blob/master/rbac_api.go): a more friendly API for RBAC. This API is a subset of Management API. The RBAC users could use this API to simplify the code. See [here](https://github.com/casbin/casbin/blob/master/rbac_api_test.go) for examples.

We also provide a web-based UI for model management and policy management:

![model editor](https://hsluoyz.github.io/casbin/ui_model_editor.png)

![policy editor](https://hsluoyz.github.io/casbin/ui_policy_editor.png)

## Policy persistence

https://casbin.org/docs/en/adapters

## Policy consistence between multiple nodes

https://casbin.org/docs/en/watchers

## Role manager

https://casbin.org/docs/en/role-managers

## Benchmarks

https://casbin.org/docs/en/benchmark

## Examples

| Model                     | Model file                                                                                                                       | Policy file                                                                                                                      |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| ACL                       | [basic_model.conf](https://github.com/casbin/casbin/blob/master/examples/basic_model.conf)                                       | [basic_policy.csv](https://github.com/casbin/casbin/blob/master/examples/basic_policy.csv)                                       |
| ACL with superuser        | [basic_model_with_root.conf](https://github.com/casbin/casbin/blob/master/examples/basic_with_root_model.conf)                   | [basic_policy.csv](https://github.com/casbin/casbin/blob/master/examples/basic_policy.csv)                                       |
| ACL without users         | [basic_model_without_users.conf](https://github.com/casbin/casbin/blob/master/examples/basic_without_users_model.conf)           | [basic_policy_without_users.csv](https://github.com/casbin/casbin/blob/master/examples/basic_without_users_policy.csv)           |
| ACL without resources     | [basic_model_without_resources.conf](https://github.com/casbin/casbin/blob/master/examples/basic_without_resources_model.conf)   | [basic_policy_without_resources.csv](https://github.com/casbin/casbin/blob/master/examples/basic_without_resources_policy.csv)   |
| RBAC                      | [rbac_model.conf](https://github.com/casbin/casbin/blob/master/examples/rbac_model.conf)                                         | [rbac_policy.csv](https://github.com/casbin/casbin/blob/master/examples/rbac_policy.csv)                                         |
| RBAC with resource roles  | [rbac_model_with_resource_roles.conf](https://github.com/casbin/casbin/blob/master/examples/rbac_with_resource_roles_model.conf) | [rbac_policy_with_resource_roles.csv](https://github.com/casbin/casbin/blob/master/examples/rbac_with_resource_roles_policy.csv) |
| RBAC with domains/tenants | [rbac_model_with_domains.conf](https://github.com/casbin/casbin/blob/master/examples/rbac_with_domains_model.conf)               | [rbac_policy_with_domains.csv](https://github.com/casbin/casbin/blob/master/examples/rbac_with_domains_policy.csv)               |
| ABAC                      | [abac_model.conf](https://github.com/casbin/casbin/blob/master/examples/abac_model.conf)                                         | N/A                                                                                                                              |
| RESTful                   | [keymatch_model.conf](https://github.com/casbin/casbin/blob/master/examples/keymatch_model.conf)                                 | [keymatch_policy.csv](https://github.com/casbin/casbin/blob/master/examples/keymatch_policy.csv)                                 |
| Deny-override             | [rbac_model_with_deny.conf](https://github.com/casbin/casbin/blob/master/examples/rbac_with_deny_model.conf)                     | [rbac_policy_with_deny.csv](https://github.com/casbin/casbin/blob/master/examples/rbac_with_deny_policy.csv)                     |
| Priority                  | [priority_model.conf](https://github.com/casbin/casbin/blob/master/examples/priority_model.conf)                                 | [priority_policy.csv](https://github.com/casbin/casbin/blob/master/examples/priority_policy.csv)                                 |

## Middlewares

Authz middlewares for web frameworks: https://casbin.org/docs/en/middlewares

## Our adopters

https://casbin.org/docs/en/adopters

## License

This project is licensed under the [Apache 2.0 license](LICENSE).
