# Java controller constraints
If you like annotations in Java code, you're in for a treat.  If you don't, this may be a good time to consider the Scala version.

One very important point to bear in mind is the order in which Play evaluates annotations.  Annotations applied to a method are applied to annotations applied to a class.  This can lead to situations where Deadbolt method constraints deny access because information from a class constraint.  See the section on deferring method-level interceptors for a solution to this.

As with the previous chapter, here is a a breakdown of all the Java annotation-driven interceptors available in Deadbolt Java, with parameters, usages and tips and tricks.

## Static constraints
Static constraints, such as `Restrict`, are implemented entirely within Deadbolt because it can finds all the information needed to determine authorization automatically.  For example, if a constraint requires two roles, "foo" and "bar" to be present, the logic behind the `Restrict` constraint knows it just needs to check the roles of the current subject.

#### SubjectPresent
`SubjectPresent` is one of the simplest constraints offered by Deadbolt.  It checks if there is a subject present, by invoking `DeadboltHandler#getSubject` and allows access if the result is an `Optional` containing a value.

##### Scope

`@SubjectPresent` can be used at the class or method level.

##### Parameters

|Parameter                |Type                              |Default             |Notes                                             |
|-------------------------|----------------------------------|--------------------|--------------------------------------------------|
| content                 | String                           |""                  | A hint to indicate the content  expected in      |
|                         |                                  |                    | in the response.  This value will be passed      |
|                         |                                  |                    | to `DeadboltHandler#onAccessFailure`.   The      |
|                         |                                  |                    | value of this parameter is completely arbitrary. |
|-------------------------|----------------------------------|--------------------|--------------------------------------------------|
| handlerKey              | String                           |"defaultHandler"    | The name of a handler in the `HandlerCache`      |
|-------------------------|----------------------------------|--------------------|--------------------------------------------------|
| deferred                | boolean                          |false               | If true, the interceptor will not be applied     |
|                         |                                  |                    | until a `DeadboltDeferred` annotation is         |
|                         |                                  |                    | encountered.                                     |

##### Example 1. Deny access to any method of a controller unless there is a user present

    @SubjectPresent
    public class MyController extends Controller
    {
        public F.Promise<Result> index()
        {
            // this method will not be invoked unless there is a subject present
            ...
        }

        public F.Promise<Result> search()
        {
            // this method will not be invoked unless there is a subject present
            ...
        }
    }


##### Example 2. Deny access to a single method of a controller unless there is a user present

    public class MyController extends Controller
    {
        public F.Promise<Result> index()
        {
            // this method is accessible to anyone
            ...
        }

        @SubjectPresent
        public F.Promise<Result> search()
        {
            // this method will not be invoked unless there is a subject present
            ...
        }
    }

 {pagebreak} 

#### SubjectNotPresent
`SubjectNotPresent` is the opposite in functionality of `SubjectPresent`.  It checks if there is a subject present, by invoking `DeadboltHandler#getSubject` and allows access only if the result is an empty `Optional`.

##### Scope

`@SubjectNotPresent` can be used at the class or method level.

##### Parameters

|Parameter                |Type                              |Default             |Notes                                             |
|-------------------------|----------------------------------|--------------------|--------------------------------------------------|
| content                 | String                           |""                  | A hint to indicate the content  expected in      |
|                         |                                  |                    | in the response.  This value will be passed      |
|                         |                                  |                    | to `DeadboltHandler#onAccessFailure`.   The      |
|                         |                                  |                    | value of this parameter is completely arbitrary. |
|-------------------------|----------------------------------|--------------------|--------------------------------------------------|
| handlerKey              | String                           |"defaultHandler"    | The name of a handler in the `HandlerCache`      |
|-------------------------|----------------------------------|--------------------|--------------------------------------------------|
| deferred                | boolean                          |false               | If true, the interceptor will not be applied     |
|                         |                                  |                    | until a `DeadboltDeferred` annotation is         |
|                         |                                  |                    | encountered.                                     |

##### Example 1. Deny access to any method of a controller if there is a user present

    @SubjectNotPresent
    public class MyController extends Controller
    {
        public F.Promise<Result> index()
        {
            // this method will not be invoked if there is a subject present
            ...
        }

        public F.Promise<Result> search()
        {
            // this method will not be invoked if there is a subject present
            ...
        }
    }


##### Example 2. Deny access to a single method of a controller if there is a user present

    public class MyController extends Controller
    {
        public F.Promise<Result> index()
        {
            // this method is accessible to anyone
            ...
        }

        @SubjectNotPresent
        public F.Promise<Result> search()
        {
            // this method will not be invoked unless there is not a subject present
            ...
        }
    }

 {pagebreak} 

#### Restrict
The Restrict constraint requires that a) there is a subject present, and b) the subject has ALL the roles specified in the at least one of the `Group`s in the constraint.  The key thing to remember about ´Restrict´ is that it ANDs together the role names within a group and ORs between groups.

##### Notation
The role names specified in the annotation can take two forms.

1. Exact form - the subject must have a role whose name matches the required role exactly.  For example, for a constraint `@Restrict("foo")` the Subject *must* have a `Role` whose name is "foo".
2. Negated form - if the required role starts starts with a !, the constraint is negated.  For example, for a constraint `@Restrict("!foo")` the Subject *must not* have a `Role` whose name is "foo".

##### Scope

`@Restrict` can be used at the class or method level.

##### Parameters

|Parameter                |Type                              |Default                |Notes                                             |
|-------------------------|----------------------------------|-----------------------|--------------------------------------------------|
| value                   | Group[]                          | N/A                   | For each `Group`, the roles that must (or in the |
|                         |                                  |                       | case of negation, must not) be held by the       |
|                         |                                  |                       | `Subject`.  When the restriction is applied, the |
|                         |                                  |                       | `Group` instances are OR'd together.             |
|-------------------------|----------------------------------|-----------------------|--------------------------------------------------|
| content                 | String                           | ""                    | A hint to indicate the content  expected in      |
|                         |                                  |                       | in the response.  This value will be passed      |
|                         |                                  |                       | to `DeadboltHandler#onAccessFailure`.   The      |
|                         |                                  |                       | value of this parameter is completely arbitrary. |
|-------------------------|----------------------------------|-----------------------|--------------------------------------------------|
| handlerKey              | String                           | "defaultHandler"      | The name of a handler in the `HandlerCache`      |
|-------------------------|----------------------------------|-----------------------|--------------------------------------------------|
| deferred                | boolean                          | false                 | If true, the interceptor will not be applied     |
|                         |                                  |                       | until a `DeadboltDeferred` annotation is         |
|                         |                                  |                       | encountered.                                     |

##### Example 1. For every action in a controller, require the `Subject` to have *editor* and *viewer* roles

    @Restrict(@Group{"editor", "viewer"})
    public class MyController extends Controller
    {
        public F.Promise<Result> index()
        {
            // this method will not be invoked unless the subject has editor and viewer roles
            ...
        }

        public F.Promise<Result> search()
        {
            // this method will not be invoked unless the subject has editor and viewer roles
            ...
        }
    }

##### Example 2. Have different requirements for each action in the controller.

    public class MyController extends Controller
    {
        @Restrict(@Group("editor"))
        public F.Promise<Result> edit()
        {
            // this method will not be invoked unless the subject has editor role
            ...
        }

        @Restrict(@Group("view"))
        public F.Promise<Result> view()
        {
            // this method will not be invoked unless the subject has viewer role
            ...
        }
    }

##### Example 3. Ensure the `Subject` has the *editor* but not the *viewer* role

    @Restrict(@Group({"editor", "!viewer"}))
    public class MyController extends Controller
    {
        public F.Promise<Result> edit()
        {
            // this method will not be invoked unless the subject has editor role AND does not have the viewer role
            ...
        }

        public F.Promise<Result> view()
        {
            // this method will not be invoked unless the subject has editor role AND does not have the viewer role
            ...
        }
    }

##### Example 4. Require the `Subject` to have *editor* or *viewer* roles

    @Restrict({@Group("editor"), @Group("viewer")})
    public class MyController extends Controller
    {
        public F.Promise<Result> index()
        {
            // this method will not be invoked unless the subject has editor or viewer roles
            ...
        }

        public F.Promise<Result> search()
        {
            // this method will not be invoked unless the subject has editor or viewer roles
            ...
        }
    }

##### Example 5. Require the `Subject` to have *customer* AND "viewer", OR "support" AND *viewer* roles

    @Restrict({@Group({"customer", "viewer"}), @Group({"support", "viewer"})})
    public class MyController extends Controller
    {
        public F.Promise<Result> index()
        {
            // this method will not be invoked unless the subject has customer and viewer,
            // or support and viewer roles
            ...
        }
    }

##### Example 6. Require the `Subject` to have *customer* AND NOT "viewer", OR "support" AND NOT *viewer* roles

    @Restrict({@Group("customer", "!viewer"), @Group("support", "!viewer")})
    public class MyController extends Controller
    {
        public F.Promise<Result> index()
        {
            // this method will not be invoked unless the subject has customer but not viewer roles,
            // or support but not viewer roles
            ...
        }
    }

 {pagebreak}

#### Unrestricted
`Unrestricted` allows you to over-ride more general constraints, i.e. if a controller is annotated with `@SubjectPresent` but you want an action in there to be accessible to everyone.

    @SubjectPresent
    public class MyController extends Controller
    {
        public F.Promise<Result> foo()
        {
            // a subject must be present for this to be accessible
        }
        
        @Unrestricted
        public F.Promise<Result> bar()
        {
            // anyone can access this action
        }
    }

You can also flip this on its head, and use it to show that a controller is explicitly unrestricted - used in this way, it's a marker of intent rather than something functional.  Because method-level constraints are evaluated first, you can have still protected actions within an `@Unrestricted` controller.

    @Unrestricted
    public class MyController extends Controller
    {
        @SubjectPresent
        public F.Promise<Result> foo()
        {
            // a subject must be present for this to be accessible
        }
        
        public F.Promise<Result> bar()
        {
            // anyone can access this action
        }
    }

##### Scope
`@Unrestricted` can be used at the class or method level.

##### Parameters

|Parameter                |Type                              |Default                |Notes                                             |
|-------------------------|----------------------------------|-----------------------|--------------------------------------------------|
| content                 | String                           | ""                    | A hint to indicate the content  expected in      |
|                         |                                  |                       | in the response.  This value will be passed      |
|                         |                                  |                       | to `DeadboltHandler#onAccessFailure`.   The      |
|                         |                                  |                       | value of this parameter is completely arbitrary. |
|-------------------------|----------------------------------|-----------------------|--------------------------------------------------|
| handlerKey              | String                           | "defaultHandler"      | The name of a handler in the `HandlerCache`      |
|-------------------------|----------------------------------|-----------------------|--------------------------------------------------|
| deferred                | boolean                          | false                 | If true, the interceptor will not be applied     |
|                         |                                  |                       | until a `DeadboltDeferred` annotation is         |
|                         |                                  |                       | encountered.                                     |

 {pagebreak} 

## Dynamic constraints
Dynamic constraints are, as far as Deadbolt is concerned, completely arbitrary.  The logic behind the constraint comes entirely from a third party.

 {pagebreak} 

#### Dynamic
todo

 {pagebreak} 

#### Pattern
todo

 {pagebreak} 
 
## Deferring method-level annotation-driven interceptors
Play executes method-level annotations before controller-level annotations. This can cause issues when, for example, you want a particular action to be applied for method and before the method annotations. A good example is `Security.Authenticated(Secured.class)`, which sets a user’s user name for `request().username()`. Combining this with method-level annotations that require a user would fail, because the user would not be present at the time the method interceptor is invoked.

One way around this is to apply `Security.Authenticated` on every method, which violates DRY and causes bloat.

    public class DeferredController extends Controller
    {
        @Security.Authenticated(Secured.class)
        @Restrict(value="admin")
        public F.Promise<Result> someAdminFunction()
        {
            return ok(accessOk.render());
        }

        @Security.Authenticated(Secured.class)
        @Restrict(value="editor")
        public F.Promise<Result> someEditorFunction()
        {
            return ok(accessOk.render());
        }
    }

A better way is to set the `deferred` parameter of the Deadbolt annotation to `true`, and then use `@DeferredDeadbolt` at the controller level to execute the method-level annotations at controller-annotation time. Since annotations are processed in order of declaration, you can specify `@DeferredDeadbolt` after `@Security.Authenticated` and so achieve the desired effect.

    @Security.Authenticated(Secured.class)
    @DeferredDeadbolt
    public class DeferredController extends Controller
    {
        @Restrict(value="admin", deferred=true)
        public F.Promise<Result> someAdminFunction()
        {
            return ok(accessOk.render());
        }

        @Restrict(value="admin", deferred=true)
        public F.Promise<Result> someAdminFunction()
        {
            return ok(accessOk.render());
        }
    }

Specifying a controller-level restriction as `deferred` will work, if the annotation is higher in the annotation list than `@DeferredDeadbolt` annotation, but this is essentially pointless.  If your constraint is already at the class level, there's no need to defer it.  Just ensure it appears below any other annotations you wish to have processed first.

## Invoking DeadboltHandler#beforeAuthCheck independently
`DeadboltHandler#beforeAuthCheck` is invoked by each interceptor prior to running the actual constraint tests.  If the method call returns an empty `Optional`, the constraint is applied, otherwise the contained in the `Optional` result is returned.  The same logic can be invoked independently, using the `@BeforeAccess` annotation, in which case the call to `beforeRoleCheck` itself becomes the constraint.  Or, to say it in code,

    result = preAuth(true, ctx, deadboltHandler)
              .flatMap(preAuthResult -> preAuthResult.map(F.Promise::pure)
                                                     .orElseGet(() -> delegate.call(ctx))

##### Scope

`@BeforeAccess` can be used at the class or method level.

##### Parameters

|Parameter                |Type                              | Default               |Notes                                             |
|-------------------------|----------------------------------|-----------------------|--------------------------------------------------|
| handlerKey              | String                           | "defaultHandler"      | The name of a handler in the `HandlerCache`      |
|-------------------------|----------------------------------|-----------------------|--------------------------------------------------|
| alwaysExecute           | boolean                          | true                  | By default, if another Deadbolt action has       |
|                         |                                  |                       | already been executed in the same request and has|
|                         |                                  |                       | allowed access, `beforeAuthCheck` will not be    |
|                         |                                  |                       | executed again.  Set this to true if you want it |
|                         |                                  |                       | to execute unconditionally.                      |
|-------------------------|----------------------------------|-----------------------|--------------------------------------------------|
| deferred                | boolean                          | false                 | If true, the interceptor will not be applied     |
|                         |                                  |                       | until a DeadboltDeferred annotation is           |
|                         |                                  |                       | encountered.                                     |

## Customising the inputs of annotation-driven actions
One of the problems with Deadbolt's annotations is they require strings to specify, for example, role names or pattern values.  It would be far safer to use enums, but this is not possible for a module - it would completely kill the generic applicability of the annotations.  If Deadbolt shipped with an enum containing roles, how would you extend it?  You would be stuck with whatever was specified, or forced to fork the codebase and customise it.  Similarly, annotations can neither implement, extend or be extended.

To address this situation, Deadbolt has three constraints whose inputs can be customised to some degree.  The trick lies, not with inheritence, but delegation and wrapping.  The constraints are

 * Restrict
 * Dynamic
 * Pattern

Here, I'll explain how to customise the Restrict constraint to use enums as annotation parameters, but the principle is the same for each constraint.

To start, create an enum that represents your roles.

    public enum MyRoles implements Role
    {
        foo,
        bar,
        hurdy;

        @Override
        public String getRoleName()
        {
            return name();
        }
    }

To allow the AND/OR syntax that `Restrict` uses, another annotation to group them together is required.

    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface MyRolesGroup
    {
        MyRoles[] value();
    }

Next, create a new annotation to drive your custom version of Restrict.  Note that an array of `MyRoles` values can be placed in the annotation.  The standard `Restrict` annotation is also present to provide further configuration.  This means your customisations are minimised. 

    @With(CustomRestrictAction.class)
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ElementType.METHOD, ElementType.TYPE})
    @Documented
    @Inherited
    public @interface CustomRestrict
    {
        MyRolesGroup[] value();

        Restrict config();
    }

The code above contains `@With(CustomRestrictAction.class)` in order to specify the action that should be triggered by the annotation.  This action can be implemented as follows. 

    @Override
    public Result call(Http.Context context) throws Throwable
    {
        final CustomRestrict outerConfig = configuration;
        RestrictAction restrictAction = new RestrictAction(configuration.config(),
                                                           this.delegate)
        {
            @Override
            public List<String[]> getRoleGroups()
            {
                final List<String[]> roleGroups = new ArrayList<>();
                for (MyRolesGroup mrg : outerConfig.value())
                {
                    final List<String> group = new ArrayList<>();
                    for (MyRoles role : mrg.value())
                    {
                        group.add(role.getName());
                    }
                    roleGroups.add(group.toArray(group.size()));
                }
                return roleGroups;
            }
        };
        return restrictAction.call(context);
    }

To use your custom annotation, you apply it as you would any other Deadbolt annotation.

    @CustomRestrict(value = {MyRoles.foo, MyRoles.bar}, config = @Restrict(""))
    public static Result customRestrictOne() {
        return ok(accessOk.render());
    }

Each customisable action has one or more extension points.  These are

|Action class       |Extension points                  |
|-------------------|----------------------------------|
|RestrictAction     | * List<String[]> getRoleGroups() |
|-------------------|----------------------------------|
|DynamicAction      | * String getValue()              |
|                   | * String getMeta()               | 
|-------------------|----------------------------------|
|PatternAction      | * String getValue()              |