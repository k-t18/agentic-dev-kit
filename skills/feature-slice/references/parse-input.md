# Parse input (mode B — add an endpoint)

**Do not write any files yet.** From the pasted content, extract the fields below.

## Accepted formats

| Format | Example |
| --- | --- |
| Frappe URL + JSON | `GET /api/method/{app}.api.v1.order.get_orders?trip_id=T-001` + response JSON |
| curl command | `curl -X POST .../order.create_order -d '{"trip_id":"T-001"}'` |
| Field list | `Module: order · Fn: get_orders · Params: trip_id (str) · Returns: name, customer, total` |
| Raw JSON | the `{ "message": { "data": [...] } }` blob |
| Free text | "Fetch all orders for a trip. Takes trip_id. Returns order name, customer, total." |

## Extract

| Field | From | Example |
| --- | --- | --- |
| `domain` | business noun (singular) | `order` |
| `httpMethod` | GET / POST / PUT / PATCH / DELETE | `GET` |
| `module` / `fn` | Frappe module + function (snake_case) | `order` / `get_orders` |
| `queryParams` / `requestBody` | URL params / body fields + types | `trip_id: string` |
| `responseFields` | fields inside `message.data` | `name, customer, total_amount` |
| `Resource` | PascalCase TS interface | `Order` |
| `serviceFnName` | camelCase method | `getOrders` |
| `hookName` | `use{Verb}{Resource}` | `useGetOrders` |

## httpMethod → hook

| Method | Verb | Wrapper |
| --- | --- | --- |
| GET | `Get` | `useApiQuery` |
| POST | `Create` | `useApiMutation` (online) or `usecases`+`outbox` (offline write) |
| PUT / PATCH | `Update` | `useApiMutation` / offline write |
| DELETE | `Delete` | `useApiMutation` / offline write |

## Type inference

String / unknown id → `string`; numeric → `number`; boolean → `boolean`; array →
`Element[]`; nested object → its own interface; absent in some responses → `?`. Never `any`.

## Before generating

Print a parse summary and, **if any field is ambiguous (especially `domain`/`module`),
ask — do not guess**:

```
Parsed:  domain=order · GET · module=order · fn=get_orders
         Resource=Order · hook=useGetOrders
         params: trip_id (string)   response: name (string), customer (string), total_amount (number)
```
