### Design decisions

- app is comprised of 2 parts:

1.  crud, simple crud on domain object which is usually same as database table in simple project.
2.  actions, more than just crud, includes business logic
3.  queries, various database queries for reports and data metrics, no business logic, but not related to crud

- Design architecture is divided into web framework, ie. User Interface and Core. That's so we can easily change the web framework by just copying core folder into new project and hook up into the new frameworks controllers. Controllers call actions and queries from Core

### Crud

- Crud part has following operations:
- get by id (read)
- create
- update
- destroy

which corresponds to the database operations:

- select by id
- insert
- update
- delete

- select all belongs to queries because with more data we will include pagination with which we are entering filtered data domain. Altough pagination is common in web apps we are gonna put it in queries part of the app becuase filter is a filter and it's common to have filters by other criterias.

- If there would be absolutelly no validation workflow would be:

1. get form data
2. call database operation

Since there is always some validation, or it's good practice to have one, workflow is following:

1. Get raw form data. Using validation library is not needed because we will validate field by field and cross field, inside the Core
2. validate form data and create domain object or errors. This part is done outside of web framework controller in single place so we can reuse it.
3. Return of validation is generic validation result object wich either contains validated domain object or errors. Reason is to have errors per field which we can use as feedback in forms or any other output. We don't want to use wtforms or other libraries to validate because they don't have all validations, like phonenumbers, they do not return static typed error objects and cross field validation is not smooth enough. This is inspired from functional programming where returning result object is practice instead of raising exceptions. Form libraries can be used for form rendering and html form <-> python object connection so we don't have to manually type the input names etc.
4. crud part now consists of application crud calls and database calls. Because we are not just saving form data to database, data flow is form data -> validate data -> database it's more clear to make distinction who takes what input. For app calls (service) input is form data, for database operations input is validated data. That's why we make separate functions instead of cramming everything into app service calls or in web framework controller.

### Filtering

Since our core is decoupled from user interface we cannot simply apply ad hoc filters to the queries in controllers. We apply filter inside app service layer (core part) so we need some kind of object that describes filtering criterias.

Queried data can be:

1. unfiltered (all)
   Unfiltered data is what is most commonly used as crud part in most other applications in the world. It's basically `select * from table`
2. paginated
   Many frameworks implement pagination helpers. Paginations had 2 filtering criterias: page and page size. In database it's implemented with `OFFSET` and `LIMIT` keywords.
3. filtered
   Filter has 3 components: field name, filter operator and value/values. `WHERE field_name fiedl_operator value/values`
4. sorted
   Data is always sorted. Either implicitly with whatever is default sorting of database or explicitly by defined criteria. We should ALWAYS use explicit ordering for consistency and avoiding incomplete data result. Sorting components are field name and direction. Direction can be either descending or ascending.

## Unsorted notes

### Data at boundaries

Boundary is where the application code stops and we enter another technology domain. Examples: JSON data, server rendered templates, database. Boundaries are infrastucural components of the application. To be able to use data outside of boundary we need to connect boundary technology to the programming language we choose to use for application. Many types of libraries are created to gap that bridge. ORMs -> they connect database table name and column names to the class and class properties respectively. Form libraries -> connecting form and form fields to the form class and class properties. Those libraries helps us to sort of keep everything in application code and avoid hard coding things like column names, writing SQL, form field names. Without them we are required to do some plumbing: `first_name = request.form.get('first-name)`, compared to the using form library: `first_name = form.first_name.data`. Many of those libraries are taking more responsibilities than just bridging the boundary, most often they take the responsibility of some part of business logic: validation. Hence we get fat models and fat form. This architecture design is about taking validation out of the boundary and doing in the application logic.

### Validation

There should be one and only one place to validate everything.

### Philosophy

1. One responsibility, one place
2. Extremly explicit
3. Use SQL
4. Code generation over code abstractions

### Read and write

Read goes directly from database mapped into the record type. Raw sql -> record. Each query has it's own result type. We do not have to deal with uneeded columns and we can use any kind of shape we need which is used directly in application code.

Writes are more complicated beacuse we have id which is generated by database, timestamps which are usually generated by ORMs. So to be completelly precise we need type for inserts and updates for the functions exposed to the app service. Inside the database module we can have internal insert and updates functions that take complete table with all the columns so it's extremely explicit. Reads are easy, but writes are more complicated. Still need time to crystalize pattern.
