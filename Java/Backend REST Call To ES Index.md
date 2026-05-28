This is an example REST call making a request to an Elasticsearch index

    @RequestMapping(value = "/api/report/users", method = RequestMethod.POST, produces = "application/json")
    @PreAuthorize("hasAnyRole('role_name')")
    public ResponseEntity<Object> runSearchForUsers(@RequestBody GridGetRowsRequestDTO aGridRequestDTO) throws IOException, ExecutionException, InterruptedException {

        if (aGridRequestDTO.getStartRow() >= aGridRequestDTO.getEndRow() ) {
            // This is an invalid request
            return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                    .contentType(MediaType.TEXT_PLAIN)
                    .body(END_ROW_GREATER_THAN_START_ROW_ERR_MSG);
        }

        // Get only records where user_type = 1
        String defaultQueryString = "user_type_id:" + 1 +"";

        // Limit the search results based on the user's organizations
        defaultQueryString = defaultQueryString + " AND (" + generateSearchQueryForUsersOrgs() + ")";


        // Change the sort field from "priority" to "priority.sort"  (so the sort is case-insensitive)
        changeSortFieldToUseElasticFieldsForSorting(aGridRequestDTO, LocalConstants.ES_USER_ID);

        setDefaultSorting(aGridRequestDTO, LocalConstants.ES_USER_ID, SORT_DESCENDING);

        // Run the search and generate a GridGetRowsResponseDTO object
        GridGetRowsResponseDTO responseDTO = serverSideGridService.getPageOfData(Constants.USERS_ES_MAPPING,
                this.esUsersReportFieldsToSearch,
                this.esUsersReportFieldsToReturn,
                defaultQueryString,
                aGridRequestDTO);

        // Return the GridGetRowsResponseDTO object and a 200 status code
        return ResponseEntity
                .status(HttpStatus.OK)
                .body(responseDTO);
    }

[generateSearchQueryForUsersOrgs](../Elasticsearch/Refine ES Query.md)