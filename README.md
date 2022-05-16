## Smartphone internet shop

This project is a React single-page application, was made with such technologies stack as:

- **React JS**
- **React Router**
- **React Redux**
- **Ramda functional library**
- **Bootstrap**

Application consists of `sticked header` and three main routes/pages:

- <ins>Main phones page
- <ins>Phone details page
- <ins>Shopping cart/basket page

These are rendered in the main root in src/index.js, inside of `<Connected router>` and Redux store `<Provider>` components. `React.lazy()` and third-party `loading spinner` are used for app optimisation, by loading only the currently used routes.

Phone data is mock and stored as an array of objects in the root folder, from where it then get fetched by fetchPhones API function, thereafter stored in Redux store and used in the project.

Store is created with `createStore()` and `createRootReducer()` as an argument, which is a result of calling `combineReducers()` function, which combines `phones`, `phonesPage`, `phonePage`, `basket` and `categories` reducer functions.

Reducer actions are described in separate `actions` folder.

### <ins>**Main phones page**</ins>

React class component `<Phones>`, rendered as one of the routes by `'/'` or category-filtered `'/categories/:id'` URLs.

Has the following structure:

- Sidebar
  - Search panel
  - Brand categories filter
- Phones card grid
  - Price sorting button
  - Phone information cards

Component uses `connect()` to get data from Redux store, through `mapDispatchToProps()` it gets `fetchPhones`, `loadMorePhones`, `addPhoneToBasket` and `fetchCategories` reducer action functions to its props.

`componentDidMount` then calls *`fetchPhones`* and *`fetchCategories`*.

*`fetchPhones`* is a reducer action function, it calls `fetchPhonesApi` to get **`phones`** array by fetching data from mockPhones array, then it dispatches `FETCH_PHONES` action and saves **`phones`** array in the payload.

`phones` reducer function then catches `FETCH_PHONES` action, with Ramda `indexBy()` function turns a list of phone objects into an object indexing the objects by `'id'`, then merges this object with reducer state by using Ramda `merge()` immutable function.

The same time, `phonesPage` reducer also catches `FETCH_PHONES` action, picks phones IDs with Ramda `pluck()` and merges this IDs object with its state, for further filtering and indexing purposes by using IDs.

*`fetchCategories`* works in the same way as `fetchPhones`, just fetching mockCategories instead of `phones`, then by the same way processes and stores **`categories`** in categories reducer state.

Then through `mapStateToProps()`, main `<Phones>` component gets **`phones`** array in its props by using *`getPhones()`* selector function.

*`getPhones()`* is responsible for extracting 'phones-related' state from `store`, filtering and searching this data by IDs, category, user input search.

What it does:

- initially extracts **`phonesPage`** reducer state from `store`, **`phonesPage`** is an object with `phones IDs` and `search` text value, which gets its value from the `<Sidebar>` component's `<Search>` input field
- uses Ramda `compose()` to:
  - `map()` `'phonesPage.ids'` and use received phone ID to find phones in previously fetched **`phones`** reducer state by these IDs
  - then use Ramda `filter()` to filter phones by selected category, by comparing mapped `phone` item's `'categoryId'` with **`activeCategoryId`**, which it gets from `ownProps`, previously passed from `<Phones>` `mapStateToProps`
  - Ramda `filter()` to filter phones by user-inputted search text, which it gets from `'phonesPage.search'` prop, and compares it with mapped `phone` item's `'name'` prop, by using Ramda `includes()` and `toLowercase` both compared values.
- Saves the result of Ramda `compose()` to new array **`phones`** and returns it

On initial render this array is equal to original phones array as no filtering is applied.

`<Phones>` component then receives this array in its props, then `componentDidUpdate` stores `props.phones` in component `state.phones`.

This `'state.phones'` then used to render initial phone cards grid.

**Sidebar**

React component, rendered in `<Layout>` component, which is a part of main `<Phones>` page. Consists of `<Search>` and `<Categories>` components.

<ins>Search panel

Class component with a state, allows to do a quick search by inputting phone name, fully or partionally, upper or lowercase.

Component uses `connect()` to get data from Redux store, through `mapDispatchToProps()` it gets *`searchPhone`* reducer action function to its props.

Component renders form element with input field, input has a change listener, which saves inputted value in state, form has a submit listener, which uses this value from a state as an argument while calling *`searchPhone`* action function from props.

*`searchPhone`* action function then dispatches `SEARCH_PHONE` action and saves text value from its argument in the payload.

`phonesPage` reducer catches `SEARCH_PHONE` action, and merges its `'state.search'` with action payload.

It makes the `store` to change, and its change makes `<Phones>` component to re-render, it calls *`getPhones()`* function again.

*`getPhones()`* uses its filtering functionality, to return only the phones from the original reducer `phones.state` which names fit the user search input value, `<Phones>` component then gets new array in its state and re-renders phones grid with it.

<ins>Categories panel

`<Categories>` component, panel with different `phone` brand name categories, allows user to filter phones grid by selecting particular category.

Uses `connect()`, gets **`categories`** and **`activeCategoryId`** data from `store` to its props through `mapStateToProps`

**`categories`** prop is a result of calling *`getCategories`* selector function, which returns an array of categories names by calling a method Ramda `values()` on `categories` reducer state, which itself is an array of previously fetched categories from mockCategories.

**`activeCategoryId`** prop is a result of calling *`getActiveCategoryId(ownProps)`* with ownProps passed from component `mapStateToProps`

*`getActiveCategoryId(ownProps)`* returns a category ID by calling Ramda `path()` on ownProps.

In `<Categories>` component then, **`categories`** prop gets `mapped` on `category`, on every category `renderCategory` method is called, which renders a `<Link>`, leading to `'/categories/${category.id}'` and having `'category.name'` inside. `renderCategory` also compares mapped `'category.id'` and **`activeCategoryId`**, and adds `'active'` class name to the link if compared categories are similar.

`'All'` category, which shows all phones without filtering, is rendered separately and returns a `<Link>` leading to `'/'`, then both `All` and other `categories` are rendered in same block.

Selecting the category triggers `<Phones>` component to render by its `'/categories/:id'` route, this calls *`getPhones()`* function again, which uses its filtering functionality to filter phones by selected category, by comparing mapped `phone` array item's 'categoryId' with activeCategoryId, which it gets from ownProps, previously passed from `<Phones>` `mapStateToProps`.

*`getPhones()`* then returns category-filtered phones array, which is used in `<Phones>` component to render phones grid.

**Phones card grid**

Second and main part of `<Phones>` component, renders grid of phones, in the way described earlier. Itself consists of **`price sorting button`** and **`phone card`**.

**`price sorting button`** is simply a button with click listener, calls *`sort()`* function.

*`sort()`* then uses Ramda `sortWith()` to sort `<Phones>` component's `state.phones`, which is  array previously processed and returned by *`getPhones()`* selector function. `'phone.price'` prop is used for sorting, and sorting order is managed by `state.priceSortDesc` boolean.

**`phone card`** is a `<div>` element, consists of *`phone images slider`* block and *`description`* block with *`phone information button`* and *`buy phone button`*.

*`phone images slider`* is a third-party plug-in by `'pure-react-carousel'`. Gets and renders phone images from `'state.phone.images'` array, and allows user to slide or click preview image links to see phone images.

*`description`* block uses `'state.phone.description'` to render mock description.

*`phone information button`* is a `<Link>` to `'/phones/${phone.id}'` route, which takes clicked phone ID and renders `<Phone>` component for particular phone.

*`buy phone button`* is a button with a click listener, which calls local *`addPhone()`* funcion and passes `'state.phone.id'` as an argument.

*`addPhone()`* itself calls *`addPhoneToBasket(id)`* reducer action function from `<Phones>` component props, to where it's previously extracted from store by `mapDispatchToProps`.

*`addPhoneToBasket(id)`* receives clicked `'phone.id'` from *`addPhone()`* argument, dispatches `ADD_PHONE_TO_BASKET` action and passes `'phone.id'` to the payload.

`basket` reducer function then catches `ADD_PHONE_TO_BASKET` action and saves `'phone.id'` from action payload to its state and therefore global store. This value is then used in `<Basket>` component, will be described later.

Additionally `<Phones>` component has `loadMorePhones` button, which doesn't actually work and just additionally renders phones to the grid from the same `<Phones>` component `state.phones` mock data array, by dispatching `LOAD_MORE_PHONES` action and catching it in `phones` and `phonesPage` reducers, and saving additional list of IDs to the store.

### <ins>**Phone details page**</ins>

React class stateless component `<Phone>`, consists of **`phone description`** and **`sidebar`** parts.

Component uses connect, gets **`phone`** array through `mapStateToProps()` and *`fetchPhoneById`*, *`addPhoneToBasket`* action functions through `mapDispatchToProps()`.

`componentDidMount` uses URL to get the phone id and calls *`fetchPhoneById(this.props.match.params.id)`*. It calls `fetchPhoneByIdApi(id)`, which fetches `phone` with this ID from mockPhones array, then passes this `phone` to the payload and dispatches `FETCH_PHONE_BY_ID_SUCCESS` action.

`phonePage` reducer catches this action and through its state saves ID to the `store`.

**`phone`** array in `mapStateToProps()` is then a result of calling **`getPhoneById(state, state.phonePage.id)`** selector function. It uses Ramda `prop()` and `id` to find `phone` in main store `state.phones`, to where it previously got fetched by *`fetchPhoneById`*.

**Phone description**

Consists of `images slider`, `phone details`, price, name and description.

`phone details` is a result of calling `renderFields()` function, it uses Ramda `compose()` to: 
- use Ramda `pick()` to get phone object props
- use Ramda `toPairs()` to return an array with arrays of phone props and their values

Then this array gets mapped and phone details fields are rendered.

**Sidebar**

Consists of phone name and price, `get back to shop` and `add to cart` buttons.

`get back to shop` is a `<Link>` leading to `'/'` route.

`add to cart` has a click listener, which calls *`addPhone()`* function, similar to the one at `phones card grid`.

### <ins>**Shopping cart/basket page**</ins>

React component, consists of `cart items table` and `sidebar`.

Component uses `connect()`, gets **`phones`** and **`totalPrice`** through `mapStateToProps` and **`removePhoneFromBasket`**, **`basketCheckout`** and **`cleanBasket`** through `mapDispatchToProps`.

**`phones`** is a result of calling *`getBasketPhonesWithCount(state)`* selector function. It uses global store `'state.basket'`,which was previously filled with phones by `basket` reducer, by catching `ADD_PHONE_TO_BASKET` action, which can be dispatched by clicking `buy now` button in `<Phones>` component or `add to cart` button in `<Phone> page` component.

`'state.basket'` then gets processed, first by Ramda `uniq()` to remove repeated IDs if same phone was added to the cart several times, result saved in `uniqueIds` const.

Then to get the quantity of particular phone in basket, first `phoneCount()` function is used, where Ramda `compose()` is called on `'state.basket'`, to first `filter` it by ID and get the length of returned array. 

Then result of `phoneCount()` is used in `phoneWithCount(phone)`, where the value of this result is added as `count` property to `phone`.

*`getBasketPhonesWithCount(state)`* eventually returns **`phones`**, which is a result of calling Ramda `compose()` on `uniqueIds`, first it gets mapped on ID, which is used in called *`getPhoneById()`* other selector function to find particular phone, then returned array gets mapped again and `phoneWithCount(phone)` is called as a callback on each iteration, adding count prop to a `phone` object.

So returned **`phones`** is an array of non-repeated phones with count property.

**`totalPrice`** is a result of calling `getTotalBasketPrice(state)` selector function, it calls Ramda `compose()` on store `'state.basket'`, maps IDs and gets phones by *`getPhoneById()`*, picks `price` prop, sums it and returns `totalPrice` of phones in the basket.

**`phones`** then used to render table rows in basket table.

Each table row has `remove` icon button, it has a click listener, which calls **`removePhoneFromBasket`** action function from props. 

**`removePhoneFromBasket`** gets phone ID, saves it in the payload and dispatches `REMOVE_PHONE_FROM_BASKET` function.

`basket` reducer catches this action, calls Ramda `without()` and uses `payload.id` to remove phone with this id from the reducer state.

`sidebar` consists of `continue shopping`, `clean cart` and `checkout` buttons.

`continue shopping` is a `<Link>` to `'/'`.

`clean cart` has a click listener, which calls **`cleanBasket`** action function from props.

**`cleanBasket`** dispatches `CLEAN_BASKET`, which gets caught by `basket` reducer and initial empty state array is returned.

`checkout` is a dummy button, calls **`basket checkout`** action function which just alerts `JSON.stringify(phones)`.
