# SDUI Specification

## Preface

In today's rapidly evolving digital landscape, user interfaces (UI) play a critical role in creating engaging and dynamic user experiences. However, creating and maintaining UIs can be a challenging and time-consuming task for developers, especially as applications grow in complexity.

To address this challenge, a new approach to UI development has emerged, known as Server-Driven UI. This approach involves delegating the UI rendering logic to the server, which sends instructions to the client on how to construct the UI dynamically.

GraphQL has become increasingly popular as a way to implement Server-Driven UI. By leveraging its type system and query capabilities, GraphQL allows developers to express complex UI components and their interactions in a concise and declarative manner.

This specification outlines a set of best practices and guidelines for building Server-Driven UI using GraphQL. By following these guidelines, developers can create scalable, performant, and maintainable Server-Driven UIs that provide rich and engaging user experiences.

This specification is intended for developers who are familiar with GraphQL and are interested in exploring Server-Driven UI as a new paradigm for building user interfaces. It assumes a basic understanding of GraphQL concepts.

----

## Specification

The SDUI Specification defines the following:

- **Components** which are types within the schema that represent something that can be rendered on the client.
- **Events** which are events that users trigger that we want to track
- **Actions** which are the steps a client app should take after a certain **Event** is triggered

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

----
# UI Events

The **`UIEventType`** enum represents a complete set of all possible events which should be emitted by the application.

This enum’s values are not defined within this spec, and should be specific to the application.

```graphql
enum UIEventType {
  # ...
}
```

For example, this enum could define the values:

- **`CLICK`**: An event that should be emitted when a component is clicked.

---

The **`UIEvent`** interface represents the common set of properties that all the events in the application should have. 

This interface defines the fields:

- **`id`**: An ID field that uniquely identifies the event.
*(non-nullable)*
- **`type`**: A enum field that specifies the type of event.
*(non-nullable)*
- **`actions`**: A list of support **`UIActionType`** values that the event can trigger.
*(nullable)*

```graphql
interface UIEvent {
  id: ID!
  type: UIEventType!
  actions: [UIActionType]
}
```

By defining a **`UIEvent`** interface, we can create more specific event types that implement this interface and add fields specific to their type.

---

The **`UIEvents`** interface is represents the common set of properties that all event boundaries in the application should have.

This interface defines the fields:

- **`id`**: An ID field that uniquely identifies the event boundary.
*(non-nullable)*
- **`events`**: A list of supported **`UIEvent`** types that can be triggered within the boundary.
*(nullable)*

```graphql
interface UIEvents {
  id: ID!
  events: [UIEvent]
}
```

# UI Actions

The **`UIActionType`** enum represents a complete set of all possible actions which the application should implement.

This enum’s values are not defined within this spec, and should be specific to the application.

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

---

The **`UIAction`** interface represents the common set of properties that all UI actions in the application should have. 

This interface defines the fields:

- **`id`**: An ID field that uniquely identifies the action.
*(non-nullable)*
- **`type`**: A enum field that specifies the type of action.
*(non-nullable)*
- **`events`**: A list of support **`UIEventType`** values that can trigger the action.
*(nullable)*

```graphql
interface UIAction {
  id: ID!
  type: UIActionType!
  events: [UIEventType]
}
```

By defining a **`UIAction`** interface, we can create more specific action types that implement this interface and add fields specific to their type.

---

The **`UIActions`** interface is represents the common set of properties that all action boundaries in the application should have.

This interface defines the fields:

- **`id`**: An ID field that uniquely identifies the action boundary.
*(non-nullable)*
- **`actions`**: A list of supported **`UIAction`** types that can be triggered within the boundary.
*(nullable)*

```graphql
interface UIActions {
  id: ID!
  actions: [UIAction]
}
```

---

For example, we can create a `**ButtonComponent**` that implements **`UIActions` & `UIEvents` & `UIComponent`** interfaces and adds a **`click`** field for the buttons click event. The **`NavigateAction`** type specifies a **`url`** field that's used to navigate to a new URL when the button is clicked.

```graphql
enum UIActionType {
  NAVIGATE
}

type NavigateAction implements UIAction {
  id: ID!
  type: UIActionType!
  events: [UIEventType]
  url: String!
}

type ClickEvent implements UIEvent {
  id: ID!
  type: UIEventType!
  actions: [UIActionType]
}

"""
A button component.
"""
type ButtonComponent implements UIActions & UIEvents & UIComponent {
  "An ID for uniquly identifying the button."
  id: ID!
  "A collection of actions the button can perform."
  actions: [UIAction]
  "A collection of events the button can emit."
  events: [UIEvent]
  "A collection of components which can be nested within the button."
  components: [UIComponent]
  "A label for the button."
  label: String!
  "A click event which can be emitted by the button."
  click: ClickEvent!
  "A navigate action which can be triggered by the click event."
  navigate: NavigateAction!
}
```

In client environments where events are propagated from child to parent we can decouple the behaviour from the component, effectively creating contextual boundaries between event triggers and event responders.

```graphql
enum UIActionType {
  NAVIGATE
}

type ClickEvent implements UIEvent {
  id: ID!
  type: UIEventType!
  actions: [UIActionType]
}

type ButtonComponent implements UIEvents & UIComponent {
  id: ID!
  events: [UIEvent]
  components: [UIComponent]
  label: String!
  click: ClickEvent!
}

type NavigateAction implements UIAction {
  id: ID!
  type: UIActionType!
  url: String!
}

type ButtonActions implements UIActions & UIComponent {
  id: ID!
  actions: [UIActionType]
  components: [UIComponent]
  navigate: NavigateAction!
}
```
