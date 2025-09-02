### Design decisions

- app is comprised of 2 parts:

1.  crud, simple crud on domain object which is usually same as database table in simple project.
2.  actions, more than just crud, includes business logic
3.  queries, various database queries for reports and data metrics, no business logic, but not related to crud

- Design architecture is divided into web framework, ie. User Interface and Core. That's so we can easily change the web framework by just copying core folder into new project and hook up into the new frameworks controllers. Controllers call actions and queries from Core

### Crud

- Crud part has following operations:
- get all (read)
- get by id (read)
- create
- update
- destroy

which corresponds to the database operations:

- select all
- select by id
- insert
- update
- delete

- If there would be absolutelly no validation workflow would be:

1. get form data
2. call database operation

Since there is always some validation, or it's good practice to have one, workflow is following:

1. Get raw form data. Using validation library is not needed because we will validate field by field and cross field, inside the Core
2. validate form data and create domain object or errors. This part is done outside of web framework controller in single place so we can reuse it.
3. Return of validation is generic validation result object wich either contains validated domain object or errors. Reason is to have errors per field which we can use as feedback in forms or any other output. We don't want to use wtforms or other libraries to validate because they don't have all validations, like phonenumbers, they do not return static typed error objects and cross field validation is not smooth enough. This is inspired from functional programming where returning result object is practice instead of raising exceptions. Form libraries can be used for form rendering and html form <-> python object connection so we don't have to manually type the input names etc.
4. crud part now consists of application crud calls and database calls. Because we are not just saving form data to database, data flow is form data -> validate data -> database it's more clear to make distinction who takes what input. For app calls (service) input is form data, for database operations input is validated data. That's why we make separate functions instead of cramming everything into app service calls or in web framework controller.
