## Create Index

    PUT /index-name

## Delete an index
    DELETE /iss-position-reports

## Perform a search in my-index
    GET /iss-position-reports/_search?q="space"

## Add document to the index following the mapping 
    POST /iss-position-reports/_doc
    {
        "info": {
        "satname": "SPACE STATION",
        "satid": 25544,
        "transactionscount": 168
        },
    "positions": [
        {
        "satlatitude": 41.57373867,
        "satlongitude": 31.24161836,
        "sataltitude": 423.2,
        "azimuth": 48.01,
        "elevation": -34.56,
        "ra": 318.21457623,
        "dec": 1.86081053,
        "timestamp": 1790612377,
        "eclipsed": false
        },
        {
        "satlatitude": 41.53740055,
        "satlongitude": 31.308719,
        "sataltitude": 423.19,
        "azimuth": 48,
        "elevation": -34.59,
        "ra": 318.24223644,
        "dec": 1.83611156,
        "timestamp": 1790612378,
        "eclipsed": false
        }
        ]
    }

## Get all documents in the specified index
    get /iss-position-reports/_search
    {
        "size": 1000,
        "query": {
        "match_all": {
            }
        }
    }

## Search the specified index looking for a document where the satid (or other attribute) is 25544 (or other specified value)
    get /iss-position-reports/_search
    {
        "query": {
        "match": {
            "info.satid": "25544"
            }
        }
    }

## Search the specified index looking for a document where the satid (or other attribute) is NOT 25544 (or other specified value)

    GET /iss-position-reports/_search
    {
        "query": {
            "bool": {
                "must_not": [
                {"match": {
                "info.satid": "25544"
                       }
                    }
                ]
            }
        }
    }

## Get a specific report by ID
    get /iss-position-reports/_doc/document-id



## Get a specific report by ID - best for multiple IDs
    GET /iss-position-reports/_search
    {
        "query": {
            "ids": {
                "values": ["dfdZ6aAB4D_PTu7kAYq3"]
            }
        }
    }




## Get the number of documents within the specified index
    get /iss-position-reports/_count



## Get all the documents from an index AND sort them by a field (positions.timestamp) descending 
### documents where that field is missing come first
    GET /iss-position-reports/_search
    {
      "query": {
            "match_all": {}
        },
    "sort": [
        {
        "positions.timestamp": {
        "order": "desc",
        "missing": "_first"
            }
        }
        ]
    }

## Delete specified data in an index
    POST /iss-position-reports/_delete_by_query
    {
        "query": {
            "match_all": {
            }
        }
    }

## Get the mapping for your index
    GET iss-position-reports/_mapping

## Get the number of documents in the index
    GET iss-position-reports/_count

## Define the mapping for an index - this can define all the mappings or just specific fields
    PUT iss-position-reports/_mapping
    {
        "properties": {
            "info": {
                "properties": {
                    "location": {
                        "type": "geo_point"
                    }
                }
            }
        }
    }

