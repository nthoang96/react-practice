# REACT SOLID Principles

## S - single-responsibility

> [!NOTE]
> A module should be responsible one, and only one, actor.

❌ Bad Practice

```js
const Products = () => {
  return (
    <div className="products">
      {products.map((product) => (
        <div key={product?.id} className="product">
          <h3>{product?.name}</h3>
          <p>${product?.price}</p>
        </div>
      ))}
    </div>
  );
};
```

✅ Good Practice

```js
const Products = () => {
  return (
    <div className="products">
      {products.map((product) => (
        <Product key={product?.id} product={product} />
      ))}
    </div>
  );
};

const Product = ({ product }) => {
  return (
    <div className="product">
      <h3>{product?.name}</h3>
      <p>${product?.price}</p>
    </div>
  );
};
```

## O - open-closed

> [!NOTE]
> software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.

❌ Bad Practice

```js
// current Button Component
const Button = ({ text, onClick }) => {
  return <button onClick={onClick}>{text}</button>;
};

// Button Component is edited by adding a new prop icon
const Button = ({ text, onClick, icon }) => {
  return (
    <button onClick={onClick}>
      <i className={icon} />
      <span>{text}</span>
    </button>
  );
};

const Home = () => {
  const handleClick = () => {};

  return (
    <div>
      <Button text="Submit" onClick={handleClick} icon="fas fa-rrow-right" />
    </div>
  );
};
```

✅ Good Practice

```js
// Existing Button functional component
const Button = ({ text, onClick }) => {
  return <button onClick={onClick}>{text}</button>;
};

// IconButton component
// ✅ Good: You have not modified anything here.
const IconButton = ({ text, icon, onClick }) => {
  return (
    <button onClick={onClick}>
      <i className={icon} />
      <span>{text}</span>
    </button>
  );
};

const Home = () => {
  const handleClick = () => {
    // Handle button click event
  };

  return (
    <div>
      <Button text="Submit" onClick={handleClick} />
      <IconButton text="Submit" icon="fas fa-heart" onClick={handleClick} />
    </div>
  );
};
```

## L - liskov substitution

> [!NOTE]
> Subtype objects should be substitutable for supertype objects

❌ Bad Practice

```js
// ⚠️ Bad Practice
// This approach violates the Liskov Substitution Principle as it modifies
// the behavior of the derived component, potentially resulting in unforeseen
// problems when substituting it for the base Select component.
const BadCustomSelect = ({ value, iconClassName, handleChange }) => {
  return (
    <div>
      <i className={iconClassName}></i>
      <select value={value} onChange={handleChange}>
        <options value={1}>One</options>
        <options value={2}>Two</options>
        <options value={3}>Three</options>
      </select>
    </div>
  );
};

const LiskovSubstitutionPrinciple = () => {
  const [value, setValue] = useState(1);

  const handleChange = (event) => {
    setValue(event.target.value);
  };

  return (
    <div>
      {/** ❌ Avoid this */}
      {/** Below Custom Select doesn't have the characteristics of base `select` element */}
      <BadCustomSelect value={value} handleChange={handleChange} />
    </div>
  );
```

✅ Good Practice

```js
// This component follows the Liskov Substitution Principle and allows the use of select's characteristics.

const CustomSelect = ({ value, iconClassName, handleChange, ...props }) => {
  return (
    <div>
      <i className={iconClassName}></i>
      <select value={value} onChange={handleChange} {...props}>
        <options value={1}>One</options>
        <options value={2}>Two</options>
        <options value={3}>Three</options>
      </select>
    </div>
  );
};

const LiskovSubstitutionPrinciple = () => {
  const [value, setValue] = useState(1);

  const handleChange = (event) => {
    setValue(event.target.value);
  };

  return (
    <div>
      {/* ✅ This CustomSelect component follows the Liskov Substitution Principle */}
      <CustomSelect
        value={value}
        handleChange={handleChange}
        defaultValue={1}
      />
    </div>
  );
};
```

## I - interface segregation

> [!NOTE]
> No code should be forced to depend on methods it does not use

❌ Bad Practice

```js
// We only need two pieces informations: imageURL and name, but we pass the entire product into the component
const ProductThumbnailURL = ({ product }) => {
  return (
    <div>
      <img src={product.imageURL} alt={product.name} />
    </div>
  );
};

const Products = ({ product }) => {
  return (
    <div>
      <ProductThumbnailURL product={product} />
      <h4>{product?.name}</h4>
      <p>{product?.description}</p>
      <p>{product?.price}</p>
    </div>
  );
};

const Products = () => {
  return (
    <div>
      {products.map((product) => (
        <Product key={product.id} product={product} />
      ))}
    </div>
  );
};
```

✅ Good Practice

```js
const ProductThumbnailURL = ({ imageURL, alt }) => {
  return (
    <div>
      <img src={imageURL} alt={alt} />
    </div>
  );
};

const Products = ({ product }) => {
  return (
    <div>
      <ProductThumbnailURL imageURL={product.imageURL} alt={product.name} />
      <h4>{product?.name}</h4>
      <p>{product?.description}</p>
      <p>{product?.price}</p>
    </div>
  );
};

const Products = () => {
  return (
    <div>
      {products.map((product) => (
        <Product key={product.id} product={product} />
      ))}
    </div>
  );
};
```

## D - dependency inversion

> [!NOTE]
> No component or function should care about how a particular thing is done

❌ Bad Practice

```js
import React from "react";

const REMOTE_URL = "https://jsonplaceholder.typicode.com/users";

export const Users = () => {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch(URL)
      .then((response) => response.json())
      .then((json) => setUsers(json));
  }, []);

  return (
    <>
      <div> Users List</div>
      {filteredUsers.map((user) => (
        <div>{user.name}</div>
      ))}
    </>
  );
};
```

✅ Good Practice

```js
// ./useFetch.ts
import { useState } from "react";

export const useFetch = (URL) => {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch(URL)
      .then((response) => response.json())
      .then((json) => setData(json));
  }, []);

  return data;
};

// ./Users.tsx
import React from "react";
import useFetch from "./useFetch";

const REMOTE_URL = "https://jsonplaceholder.typicode.com/users";

export const Users = () => {
  const users = useFetch(REMOTE_URL);

  return (
    <>
      <div> Users List</div>
      {filteredUsers.map((user) => (
        <div>{user.name}</div>
      ))}
    </>
  );
};
```
