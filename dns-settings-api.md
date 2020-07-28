# Public Rest API for Namebase DNS Settings
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

# DNS Settings API

## Blockchain Settings
Blockchain settings will not update on Handshake immediately. Until they are mined, the settings will exist in two states: the current Namebase records and the current Handshake records. In the responses, the `upToDate` field will indicate if Handshake is fully synced with the Namebase records. Otherwise, `records` will just return the Namebase records.

### Get Settings
```
GET /api/v0/dns/domains/:domain
```
Returns the current blockchain DNS settings for a `domain`.

#### Response:
```
{
  "success": boolean,
  "currentHeight": integer,
  "upToDate": boolean,
  "canUseSimpleUi": boolean, // false if the records were set using the advanced settings
  "rawNameState": string,
  "fee": string,
  "records": Record[],
}
```

##### Record:
```
{
  "type": string,
  "host": string,
  "value": string,
  "ttl": integer,
}
```

### Change Settings
```
PUT /api/v0/dns/domains/:domain
```
Updates the Handshake DNS settings

#### Parameters:
| Name            | Type     | Mandatory  | Description  |     
|-----------------|----------|------------|--------------|
| `records`       | Record[] | YES        |  |

#### Record:
| Name            | Type    | Mandatory | Description                       |
|-----------------|---------|-----------|-----------------------------------|
| `type`          | ENUM    | YES       | Valid inputs: [`DS`, `NS`, `TXT`] |
| `host`          | STRING  | YES       |  |
| `value`         | STRING  | YES       |  |
| `ttl`           | INTEGER | YES       |  |

#### Response:
```
{
  "success": boolean,
  "txHash": string,
  "rawNameState": string, // hex of synced blockchain records
  "records": Record[],
}
```

##### Record:
```
{
  "type": string,
  "host": string,
  "value": string,
  "ttl": integer,
}
```

### Change Settings (advanced)
```
PUT /api/v0/dns/domains/:domain/advanced
```
Updates the Handshake DNS settings using advanced types. Not all Handshake types are supported using json input, to include other types, create a hex of the records and use this endpoint.

#### Parameters:
| Name            | Type   | Mandatory  | Description         |
|-----------------|--------|------------|---------------------|
| `rawNameState`  | STRING | YES        | Hex of DNS records  |

#### Response:
```
{
  "success": boolean,
  "txHash": string,
  "rawNameState": string,
  "canUseSimpleUi": boolean, // false if the records were set using the advanced settings
  "records": Record[],
}
```

##### Record:
```
{
  "type": string,
  "host": string,
  "value": string,
  "ttl": integer,
}
```

## Namebase Nameserver Settings
Our nameservers will update immediately.

### Get Settings
```
GET /api/v0/dns/domains/:domain/nameserver
```

#### Response:
```
{
  "success": boolean,
  "records": Record[],
}
```

##### Record:
```
{
  "type": string,
  "host": string,
  "value": string,
  "ttl": integer,
}
```

### Change Settings
```
PUT /api/v0/dns/domains/:domain/nameserver
```

#### Parameters:
| Name            | Type           | Mandatory  | Description  |     
|-----------------|----------------|------------|--------------|
| `records`       | Record[]       | YES        | The records you want set. Keep in mind that when a new record is set, every record with the same host and type will be removed and replaced with the new record |
| `deleteRecords` | DeleteRecord[] | YES        | The record sets that you want to delete. This will remove all records with the specified host and type |

##### Record:
| Name            | Type    | Mandatory | Description                       |
|-----------------|---------|-----------|-----------------------------------|
| `type`          | ENUM    | YES       | Valid inputs: [`A`, `NS`, `CNAME`, `TXT`, `DS`, `ALIAS`] |
| `host`          | STRING  | YES       | Use `@` to set the record at the root |
| `value`         | STRING  | YES       |  |
| `ttl`           | INTEGER | YES       |  |

##### DeleteRecord:
| Name            | Type    | Mandatory | Description                       |
|-----------------|---------|-----------|-----------------------------------|
| `type`          | ENUM    | YES       | Valid inputs: [`A`, `NS`, `CNAME`, `TXT`, `DS`, `ALIAS`] |
| `host`          | STRING  | YES       | Use `@` to set the zone as the root |

#### Response:
```
{
  "success": boolean,
}
```
