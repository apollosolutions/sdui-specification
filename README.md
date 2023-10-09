# SDUI Specification

## Preface

In today's rapidly evolving digital landscape, user interfaces (UI) play a critical role in creating engaging and dynamic user experiences. However, creating and maintaining UIs can be a challenging and time-consuming task for developers, especially as applications grow in complexity.

To address this challenge, a new approach to UI development has emerged, known as Server-Driven UI (SDUI). This approach involves delegating the UI rendering logic to the server, which sends instructions to the client on how to construct the UI dynamically.

GraphQL has become increasingly popular as a way to implement Server-Driven UI. By leveraging its type system and query capabilities, GraphQL allows developers to express complex UI components and their interactions in a concise and declarative manner.

This specification outlines a set of best practices and guidelines for building Server-Driven UI using GraphQL. By following these guidelines, developers can create scalable, performant, and maintainable Server-Driven UIs that provide rich and engaging user experiences.

This specification is intended for developers who are familiar with GraphQL and are interested in exploring Server-Driven UI as a new paradigm for building user interfaces. It assumes a basic understanding of GraphQL concepts.

## Contributing

The other purpose of this specificication is to lead a community effort to document all the best practices used by many teams building SDUI apps. The Apollo team has seen and used many of these principles first hand and we can guide the community but all knowledge should be shared any other best practices that should be covered are open to contribution.

Please createa an issue or pull-request with the topic or changes you would like and we can work to document the pattern if we see it being used by multiple teams.

----

## Specification

## Scalars

Default to using `String` as the return type. While there can be scenarios where other scalars can be used, when dealing with UIs we often will be returning formatted and localized data which requires text. Consider using `String`s first and then discuss why that is not the best option for each edge cases.

## Client Conditionals

Consider, where possible, moving conditionals from the client to the server. This is a commmon place for business logic to form and become tech debt that requires updating with changing business goals and priorties.

## Nullability

Use nullability as a flag to render. If the component is present, client can render. If it is `null`, clients can continue as normal.

## UI Components

It might be your first assumption to create a `UIComponent` interface that represents the common set of properties that all UI components in the application should have. Something like this:

```graphql
# DO NOT USE THIS!
interface UIComponent {
  id: ID!
  components: [UIComponent]
}
```

However, we find that at scale this interface actually becomes unusable. If every single component implemented it your fragment selections become to large and you would never actually return this one interface as the return type in any field because it would be too complicated. Enforcing the all components types have an `id` field is also not the job of a GraphQL interface, that should be handled by linters.

Instead we reccomnd page or usecase specific unions for your components.

For example, we can create a `ProductCard` type and a `AdvertisementCard` type and return either one in our `searchResults`:

```graphql
type ProductCard {
  title: String
  price: String
  reviews: [Review]
  productData: Product
}

type AdvertisementCard {
  title: String
  link: URL
  partnerInfo: Partner
}

union SearchResult = ProductCard | AdvertisementCard

type Query {
  searchResults: [SearchResult]
}
```

By using focused unions, we can define a hierarchy of components without blowing out the tree to infitite possibilities. This enables us to create all known UI layouts but still have the option to create large union sets when needed.

## Analytics

For client-side tracking or analytics, it is common to send some data to a tracking server when a client action is performed. In these scenarios, include the complete constructed payload that should be sent in the API response and have the client send that along. Client apps do not need to understand this payload so it can be a simple `String` or `JSON` scalar.

## Events

Enums can be used to represents a set of possible events which should be emitted by the application at certain points.

This enum’s values are not defined within this spec, and should be specific to the application and its place of use.

This enum should be used when the client needs to understand which event occurred. This should be less common than analytics.

```graphql
enum UIEventType {
  # ...
}
```

For example, this enum could define the values:

- **`CLICK`**: An event that should be emitted when a component is clicked.
- **`CHECKOUT`**: An event that should be emitted when the customer navaigates to checkout.
- **`REVIEWS_OPENED`**: An event that should be emitted when the review modal is clicked.

## Actions

A **`UIActionType`** enum represents a set of all possible actions which the application should implement for a given scenario.

This enum’s values are not defined within this spec, and should be specific to the application and its place of use.

```graphql
enum UIActionType {
  # ...
}
```

For example, this enum could define the values:

- **`NAVIGATE`**: An action that should change the currently rendered view.
- **`TOGGLE`**: An action that should toggle a component.
- **`SUBMIT`**: An action that should submit a form’s data.
- **`SCROLL`**: An action that should scroll the currently rendered view to a specific coordinate.
- **`FOCUS`**: An action that should change the currently focused component.
- **`CALLBACK`**: An action that should call a component-scoped function.
- **`QUERY`**: An action that should call a query operation.
- **`REQUERY`**: An action that should re-call a query operation.
- **`MUTATE`**: An action that should call a mutation operation.
