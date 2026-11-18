# Automated API Testing Suite: Tool Shop Purchase Lifecycle

A Postman API testing collection that validates the full customer purchase workflow of the [Practice Software Testing](https://api.practicesoftwaretesting.com) Tool Shop API, from account registration to invoice creation. This project demonstrates end-to-end request chaining, automated authentication handling, runtime response validation, and dynamic test data generation.

## 🛠️ Key Testing Features Covered

- **Automated Session & Token Chaining:** Captures the `access_token` returned at login and stores it in a collection variable, so the cart and checkout requests authenticate with a Bearer token without any manual copying.
- **State Persistence Between Requests:** Saves the `cart_id` created by the cart request and reuses it in the add-to-cart and checkout steps.
- **Dynamic Test Data Generation:** Uses Postman's built-in faker variables (`{{$randomFirstName}}`, `{{$randomLastName}}`, `{{$randomStreetAddress}}`) plus pre-request scripts to generate a unique email, a throwaway password and a random Nepali phone number on every run, so repeated runs never hit a 409 conflict.
- **Address Validation Chaining:** A postcode lookup request fetches an address that the API itself recognizes and passes it to checkout, avoiding `422` city/country mismatch errors.
- **Runtime Assertions:** Post-response scripts check status codes and key response fields after each request.

## 📋 API Endpoint Coverage & Scenarios

| Request Name | Method | Endpoint | Core Validation |
|---|---|---|---|
| Registration form fillup | POST | `/users/register` | Creates a new user with generated email, phone and password |
| Login fillup | POST | `/users/login` | Validates `200 OK`, token is not empty, saves `access_token` |
| Product by Categories | GET | `/categories/tree` | Validates categories are retrieved successfully |
| Product Search for Available Tools | GET | `/products/search?q=hammer` | Validates `200 OK` and that results match the search query |
| Create an Empty cart | POST | `/carts` | Validates `201 Created`, saves `cart_id` |
| Add Product to Cart | POST | `/carts/{{cart_id}}` | Validates the item is added to the cart |
| For lookup | GET | `/postcode-lookup?country=DE&postcode=10117&house_number=1` | Validates `200 OK`, saves billing street, city and state |
| Check Out | POST | `/invoices` | Validates `201 Created` and that `invoice_number` is not empty |

## 🔗 Request Flow

```
Registration → Login → Categories → Product Search → Create Cart
→ Add Product to Cart → Postcode Lookup → Check Out
```

Each request depends on variables set by an earlier one, so the collection should be run in this order.

## 🚀 How to Import and Run This Suite

### Prerequisites
Install the desktop version of Postman, or Node.js with Newman for the command line.

### Running inside Postman
1. Clone or download this repository.
2. In Postman, click **Import** and drag in `Tool-Shop.postman_collection.json` and `Important-subjects.postman_environment.json`.
3. Select the **Important subjects** environment in the top-right dropdown. It only needs `base_url` set to `https://api.practicesoftwaretesting.com`.
4. Open the collection, click **...**, then **Run collection**, and run all requests in order.

### Running with Newman
```bash
npm install -g newman
newman run Tool-Shop.postman_collection.json -e Important-subjects.postman_environment.json
```

> **Note:** Login tokens expire after a few minutes. Run the whole collection from the start instead of sending Checkout on its own.

## 📊 Sample Assertion Snippets

```javascript
// Login: verify status and token
const res = pm.response.json();
pm.collectionVariables.set("access_token", res.access_token);

pm.test("login successful and token received", function () {
    pm.response.to.have.status(200);
    pm.expect(res.access_token).to.not.be.empty;
});

// Checkout: verify the invoice was created
pm.test("Checkout successful (201 created)", function () {
    pm.response.to.have.status(201);
    pm.expect(res.invoice_number).to.not.be.empty;
});
```

## 🔒 Security Notes

No real credentials are stored in this repository. The test password is generated at runtime and the environment file contains only the public base URL.
