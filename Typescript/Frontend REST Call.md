The frontend REST call for getting server-side data from an Elasticsearch index starts like this.  It mirrors the normal REST syntax, but the DTo used to hold the response is different.


    public runSearchForUsers(aGridGetRowsRequestDTO: GridGetRowsRequestDTO): Observable<GridGetRowsResponseDTO> {
    // Construct the URL of the REST call
    const restUrl: string = environment.baseUrl + '/api/report/users';

    // Use a POST call to send a JSON body of info
    return this.httpClient.post <GridGetRowsResponseDTO> (restUrl, aGridGetRowsRequestDTO, {} );
    }

The DTO looks like:

    export class GridGetRowsResponseDTO {
    public data: any[];
    public lastRow: number;  // If lastRow==-1, then infinite scrolling is ON.  If lastRow==totalMatches, then infinite scrolling is OFF
    public totalMatches: number;
    public secondaryColumnFields: string[];
    public searchAfterClause: string;  // Holds information about the last row so ElasticSearch can get page2, page3, ..
    public isValidQuery: boolean;
    }
