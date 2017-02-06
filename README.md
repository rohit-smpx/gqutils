# gql-utils
Utilities for GraphQL

## Functions
### `makeSchemaFromModules(modules, opts)`
Create a graphQL schema from various modules. If the module is a folder, it'll automatically require it.
```js
const modules = [
	'employee/Employee',
	'categories',
];

const schema = makeSchemaFromModules(modules, {
	baseFolder: `${__dirname}/lib`,
	allowUndefinedInResolve: false,
	resolverValidationOptions: {},
});
```

Each module can either export {schema, resolvers} or {types, queries, mutations, resolvers}.
```
const schema = /* GraphQL */`
	# @types
	# Employee
	type Employee {
		id: ID!
		smartprixId: ID
		name: String
		email: String
		personalEmail: String
		phone: String
		emergencyPhone: String
		gender: String
		dateOfBirth: String
		dateOfJoining: String
		designation: String
		aptitudeMarks: String
		bankAccountNumber: String
		panNumber: String
		status: String
		createdAt: String
		updatedAt: String
	}

	@connection(Employee)

	# @queries
	employee(
		id: ID!
		email: String
	): Employee

	employees(
		name: String
		first: Int
		last: Int
		before: String
		after: String
	): EmployeeConnection

	# @mutations
	saveEmployee(
		id: ID
		name: String
		email: String!
		personalEmail: String
		phone: String!
	): Employee

	deleteEmployee(
		id: ID!
	): DeletedItem
`;

const resolvers = {
	Query: {
		employee: getEmployee,
		employees: getEmployees,
	},
	Mutation: {
		saveEmployee,
		deleteEmployee,
	},
};

export {
	schema,
	resolvers,
};
```

In schema types, queries and mutations can be separated using `# @types`, `# @queries`, `# @mutations` to mark the begninning of each section respectively.
`@connection(typeName)` can be used to automatically generate type for a relay compatible connection for pagination.
`@connection(Employee)` will automatically generate the type `EmployeeConnection`

### `getConnectionResolver(query, args)`
Given a query (xorm query) and it's arguments, it'll automatically generate a resolver for a relay connection.
```js
async function getEmployees(root, args) {
	const query = Employee.query();
	if (args.name) {
		query.where('name', 'like', `%${args.name}%`);
	}

	return getConnectionResolver(query, args);
}
```

### `parseGraphqlSchema(schema)`
Given a schema which uses our custom schema language (having `@connection`, `# @types` etc.), it'll return {types, queries, mutations}
If you're using `makeSchemaFromModules`, you won't need to use this function.