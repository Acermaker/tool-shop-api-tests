# Tool Shop API Tests

Postman collection testing the Practice Software Testing API
(https://api.practicesoftwaretesting.com).

## How to run
1. Import both JSON files into Postman.
2. Select the "Important subjects" environment.
3. Run the collection with the Collection Runner (Run in order).

## Request order
Registration → Login → Product by Categories → Product Search →
Create an Empty cart → Add Product to Cart → For lookup → Check Out

## Notes
- Registration generates a unique email, phone and password on every run.
- "For lookup" fetches a valid billing address that Check Out uses.
- Login tokens expire after a few minutes, so run the whole collection
  instead of sending Check Out on its own.