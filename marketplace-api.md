# Public Rest API for Namebase Marketplace
## General API Information
* The root API URL is https://www.namebase.io
* Parameters for GET requests must be sent as a `query string`.
* All other request methods require parameters in the request body as `Content-Type: application/json`
* Parameter order does not matter.
* All timestamps are in milliseconds.
* API errors take the following form:
```javascript
{
  "code": "NAMEBASE_API_ERROR_CODE",
  "message": "Error message appears here."
}
```

## Rate limiting
If your account repeatedly gets rate limited, then you are using the API improperly and must back off. Failure to do so will cause your account to lose API access. Namebase retains the right to ban your account with no notice if you are abusing the API.

The rate limits are set generously and are not meant to prohibit any amount of normal usage. In the future, we will standardize our rate limits and provide more explicit feedback about how many remaining requests you can send each minute.

# Marketplace API
## Terminology
* All prices are in the units of quote asset per 1 unit of the base asset
* All quantities are in the units of the base asset, and all quote quantities are in the units of the quote asset
* Similarly, volumes are in the units of the base asset, and quote volumes are in the units of the quote asset

### Marketplace Listings
```
GET /api/domains/marketplace/:offset
```
Returns 100 sorted names, paginated by an `offset` parameter. For example, `offset=0` will get the first 100 listings and `offset=100` will return listings 101-200.

#### Parameters:
| Name            | Type   | Mandatory | Description                                                       |
|-----------------|--------|-----------|--------------------------------------------------|
| `sortKey`       | ENUM   | YES       | Valid sortKeys: [`bid`, `price`, `name`] |
| `sortDirection` | ENUM   | YES       | Valid sortDirections: [`desc`, `asc`]             |

#### Response:
```
{
  "success": boolean,
  "domains": [
    {
      "id": string,
      "amount": string,
      "name": string,
      "description": string,
      "watching": string
    },
  ],
}
```

### All Sale History
```
GET /api/domains/sold/:offset
```
Returns 100 sorted names, paginated by an `offset` parameter. For example, `offset=0` will get the first 100 listings and `offset=100` will return listings 101-200.

#### Parameters:
| Name            | Type   | Mandatory | Description                                                |
|-----------------|--------|-----------|------------------------------------------------------------|
| `sortKey`       | ENUM   | YES       | Default: `date`, Valid sortKeys: [`date`, `price`, `name`] |
| `sortDirection` | ENUM   | YES       | Default: `desc`, Valid sortDirections: [`desc, `asc`]      |

#### Response:
```
{
  "success": boolean,
  "domains": [
    {
      "id": string,
      "name": string,
      "amount": string,
      "asset": string,
      "watching": string,
      "created_at": datetime
    },
  ],
}
```

### Domain Sale History
```
GET /api/v0/marketplace/:domain/history
```

#### Response:
```
{
  "success": boolean,
  "history": [
    {
      "domain": string,
      "amount": string,
      "asset": string,
      "created_at": datetime,
    },
  ],
}
```

### List Name / Update Listing
```
POST /api/v0/marketplace/:domain/list
```

#### Parameters:
| Name          | Type   | Mandatory | Description                           |
|---------------|--------|-----------|---------------------------------------|
| `amount`      | STRING | YES       |                                       |
| `asset`       | STRING | YES       | Valid assets: [`HNS`]                 |
| `description` | STRING | YES       | 10,000 character max                  |

#### Response:
```
{
  "success": boolean,
}
```

### Cancel Listing
```
POST /api/v0/marketplace/:domain/cancel
```

#### Response:
```
{
  "success": boolean,
}
```

### Purchase Name
```
POST /api/v0/marketplace/:domain/buynow
```

#### Response:
```
{
  "success": boolean,
}
