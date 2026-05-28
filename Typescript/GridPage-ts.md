    
    @Component({
    selector: 'app-users-report',
    templateUrl: './users-report.component.html',
    styleUrls: ['./users-report.component.scss']
    })
    export class UsersReportComponent implements OnInit, OnDestroy{
    
    protected readonly Constants = Constants;
    private readonly PAGE_NAME: string = 'Agency Users';
    
    public  isValidQuery: boolean = true;
    public  rawSearchQuery: string = "";
    private textSearchInProgress: boolean = false;
    private listenForGridChanges: boolean = false;
    public  gridApi: GridApi;
    public  gridColumnApi: ColumnApi;
    public  displayedTotalMatches: string = "";
    private resizeSubject: Subject<any> = new Subject<Event>();
    public  totalMatchesOnPageLoad: number | null = null;
    private searchAfterClause: string | null;
    public  gridStatsIsLoading: boolean = false;
    
    private navbarSubscription: Subscription;
    private resizeSubscription: Subscription;
    private saveGridEventsSubscription: Subscription;
    public saveGridColumnStateEventsSubject: Subject<ColumnState[]> = new Subject();
    
    private userHasPastColumnState: boolean = false;
    
    constructor(private serverSideGridService: ServerSideGridService,
    private preferenceService: PreferenceService,
    private exportService: ExportService,
    ) {  }
    

    public ngOnInit() {

    // Listen for save-grid-column-state events
    // NOTE:  If a user manipulates the grid, then we could be sending LOTS of save-column-state REST calls
    //    	The debounceTime slows down the REST calls
    //    	The switchMap cancels previous calls
    //    	Thus, if there are lots of changes to the grid, we invoke a single REST call using the *LAST* event (over a span of 250 msecs)
    this.saveGridEventsSubscription = this.saveGridColumnStateEventsSubject.asObservable().pipe(

      debounceTime(250),     	// Wait 250 msecs before invoking REST call

      switchMap( (aNewColumnState: any) => {
        // Use the switchMap for its canceling effect:
        // On each observable, the previous observable is canceled

        // Return an observable
        // Invoke the REST call to save it to the back end
        return this.preferenceService.setPreferenceValueForPageUsingJson(Constants.COLUMN_STATE_PREFERENCE_NAME, aNewColumnState, this.PAGE_NAME)

      })
    ).subscribe();

    this.gridStatsIsLoading = true;
    
    };
    
    public ngOnDestroy(): void {
    if (this.navbarSubscription) {
    this.navbarSubscription.unsubscribe();
    }

    if (this.resizeSubscription) {
      this.resizeSubscription.unsubscribe();
    }

    if (this.saveGridEventsSubscription) {
      this.saveGridEventsSubscription.unsubscribe();
    }
    }
    
    public gridOptions: GridOptions = {
    domLayout: 'normal',            // Requires the wrapper div to have a height set *OR* a class="h-full" on it
    debug: false,
    rowModelType: 'serverSide',	    // Possible values are 'clientSide', 'infinite', 'viewport', and 'serverSide'
    pagination: false,              // Do not show the 1 of 20 of 20, page 1 of 1 (as we are doing infinite scrolling)
    cacheBlockSize: 40,             // Set the size of each block of records
    blockLoadDebounceMillis: 100,
    rowSelection: 'single',
    debounceVerticalScrollbar: true,
    overlayNoRowsTemplate: "<span class='no-matches-found-message'>No matches were found</span>",
    suppressCellFocus: true,
    suppressRowTransform: true,
    rowHeight: 32,  // Offsets the slight vertical imperfection from the cellRenderer
    suppressRowHoverHighlight: false,
    suppressCsvExport: true,
    suppressExcelExport: true,

    onSortChanged: () => {
      this.saveColumnState();
    },

    onDragStopped: () => {
      // User finished resizing or moving column
      this.saveColumnState();
    },

    onColumnVisible: () => {
      this.saveColumnState();
    },

    onColumnPinned: () => {
      this.saveColumnState();
    },
    
    }
    
    private saveColumnState(): void {
    if (this.listenForGridChanges) {
    // The grid has rendered data.  So, save the sort/column changes

      // Get the current column state
      let currentColumnState: ColumnState[] = this.gridColumnApi.getColumnState();

      // Send a message to save the current column state
      this.saveGridColumnStateEventsSubject.next(currentColumnState)
    }

    }


    
    private textFilterParams: any = {
    filterOptions: ['contains', 'notContains'],
    caseSensitive: false,
    debounceMs: 200,
    maxNumConditions: 1,
    };
    
    public defaultColDefs: ColDef = {
    sortable: true,
    resizable: true,
    floatingFilter: true,	// Causes the filter row to appear below column names
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    };
    
    
    public firstDataRendered(): void {
    // At this point, the grid is fully rendered.  So, set the flag to start saving sort/column changes
    this.listenForGridChanges = true;
    }
    
    
    /*
    * The grid calls onGridReady() once it is fully initialized.  This is the start of this page.
    *  1. Invoke a REST call to get the grid preferences
    *  2. When the REST call returns
    *      a. Configure the grid with the correct columns
    *      b. Initialize the server-side data source
    *         (which will cause the getRows() REST endpoint to be called asynchronously)
    */
    public onGridReady(aEvent: GridReadyEvent) {

    this.gridApi = aEvent.api;
    this.gridColumnApi = aEvent.columnApi;

    // Show the loading overlay
    this.gridApi.showLoadingOverlay();

    // Invoke the REST call to get past column state preference info for THIS PAGE
    this.preferenceService.getPreferenceValueForPage(Constants.COLUMN_STATE_PREFERENCE_NAME, this.PAGE_NAME).subscribe( (aPreference: GetOnePreferenceDTO) => {
      // REST call came back.  I have the grid preferences

      if (!aPreference.value) {
        // There is no past column state
        this.userHasPastColumnState = false;
      } else {
        // There is past column state
        let storedColumnStateObject = JSON.parse(aPreference.value);

        // Set the grid to use past column state
        this.gridColumnApi.applyColumnState({
          applyOrder: true,
          state: storedColumnStateObject
        });

        this.userHasPastColumnState = true;
      }
    });

    // initialize the grid *AFTER* setting the current page name (so it loads the correct grid grid column preferences)
    this.gridStatsIsLoading = false;



    // Use this to prevent grid reset spamming during page resize events
    this.resizeSubject.pipe(debounceTime(150)).subscribe(() => {
      // This code will only run once every 150ms (during page resizing)
      if (this.gridApi) {
        this.gridApi.sizeColumnsToFit();
      }
    });

    this.gridApi.setServerSideDatasource(this.serverSideDataSource);

    }
    
    public columnDefs: ColDef[] = [
    {
    field: 'user_id',
    headerName: 'ID',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: true,
    },
    {
    field: 'full_name',
    headerName: 'Full Name',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'csv_cert_username',
    headerName: 'Cert Username',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: true,
    },
    {
    field: 'email',
    headerName: 'Email',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'phone_number_displayed',
    headerName: 'Phone Number',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    headerName: 'User Type',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    cellRenderer: (params: any) => 'Government', // this report always returns Government Users - this is to match the manual CSV export
    hide: true,
    },
    {
    field: 'registration_state',
    headerName: 'Registration State',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'created_date',
    headerName: 'Created Date',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'primary_org',
    headerName: 'Primary Org',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'secondary_org',
    headerName: 'Secondary Org',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'tertiary_org',
    headerName: 'Tertiary Org',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: true,
    },
    {
    field: 'quaternary_org',
    headerName: 'Quaternary Org',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: true,
    },
    {
    field: 'quinary_org',
    headerName: 'Quinary Org',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: true,
    },
    {
    field: 'senary_org',
    headerName: 'Senary Org',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: true,
    },
    {
    field: 'is_nccs_originator',
    headerName: 'Is Originator?',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'is_nccs_reviewer',
    headerName: 'Is Reviewer?',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'is_nccs_certifier',
    headerName: 'Is Certifier?',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'is_nccs_contracting_officer',
    headerName: 'Is Contracting Officer?',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'last_updated_date',
    headerName: 'Last Updated Date',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: true,
    },
    {
    field: 'last_login_date',
    headerName: 'Last Login Date',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: false,
    },
    {
    field: 'csv_role_display_names',
    headerName: 'Roles',
    cellClass: 'grid-text-cell-format',
    filter: 'agTextColumnFilter',
    filterParams: this.textFilterParams,
    hide: true,
    },
    ]

    
    /*
    * User clicked to run a search
    *  1. Clear the grid cache
    *  2. Clear all sorting
    *  3. Clear all filters
    *  4. Force the grid to invoke the REST endpoint by calling onFilterChanged()
    */
      public runSearch(): void {
      // Set a flag indicating the text search is in progress
      this.textSearchInProgress = true;
    
    // Stop listening for column preference changes
    this.listenForGridChanges = false;

    this.clearGridCache();
    
    // Clear all sorting
    this.clearGridSorting();
    
    // Clear the filters
    this.gridApi.setFilterModel(null);
    
    // Force the grid to invoke the REST endpoint
    this.gridApi.onFilterChanged();
    
    // Stop this page from saving preferences after EVERY search
    // -- We reset these flags 500 milliseconds after the user runs a search
    setTimeout(() => {
    this.textSearchInProgress = false;
    
    // Start listening for column preference changes
    this.listenForGridChanges = true;
    }, 500);
    }
    
    public clearSearch(): void {
    // Clear the grid cache and move the vertical scrollbar to the top
    this.clearGridCache()

    // Clear all grid sorting
    this.clearGridSorting();

    // Clear all grid filters
    this.gridApi.setFilterModel(null);

    // Clear the search box
    this.rawSearchQuery = "";

    // Force the grid to run a search (so fresh data is loaded into the grid)
    this.gridApi.onFilterChanged();
    }
    
    private clearGridCache(): void {
    // Clear the cache by resetting the serverSideDataSource object
    // this.gridApi.setServerSideDatasource(this.serverSideDataSource);
    }
    
    private clearGridSorting() {
    this.gridColumnApi.applyColumnState({
    defaultState: {
    sort: null
    }
    });
    }
    
    public autoSizeGrid() {
    this.gridApi.sizeColumnsToFit();
    }
    
    
    public resetGrid(): void {
    // Reset columns is called *FIRST*  (so the default columns are visible and restored to default)
    this.gridColumnApi.resetColumnState();

    // Call sizeColumnsToFit *SECOND* (to make sure that all default columns appear and take the full grid width)
    this.gridApi.sizeColumnsToFit();

    // Clear the grid cache and move the vertical scrollbar to the top
    this.clearGridCache()

    // Clear all grid sorting
    this.clearGridSorting();

    // Clear all grid filters
    this.gridApi.setFilterModel(null);

    // Clear the search box
    this.rawSearchQuery = "";

    // Force the grid to run a search (so fresh data is loaded into the grid)
    this.gridApi.onFilterChanged();
    }
    
    /*
    * Create a server-side data source object
      *
      * The getRows() method is invoked when a user scrolls down (to get another page of rows)
      * The getRows() method is invoked when a user changes a filter
      * The getRows() method is invoked when a user changes sorting
      * The getRows() method is invoked manually when the code calls this.gridApi.onFilterChanged()
        */
        private serverSideDataSource: IServerSideDatasource = {
        getRows: (params: IServerSideGetRowsParams) => {
        // The grid needs to load data.  So, subscribe to gridService.getServerSideData() and load the data
    
        if (params.request.startRow == 0) {
        // The user is requesting a first page (so we are not getting a 2nd or 3rd page)
        // -- Reset the additional sort fields  (needed for the 2nd, 3rd, 4th pages)
        this.searchAfterClause = null;
        }
    
        // Hide any "loading" or "no results found" overlays
        this.gridApi.hideOverlay();
    
        // Add the additional sort fields to the request object 
        let getRowsRequestDTO: GridGetRowsRequestDTO = new GridGetRowsRequestDTO(params.request, this.searchAfterClause, this.rawSearchQuery)
            // The definition to this DTO is linked at the botton
 
        // Invoke the REST Call (to run the search)
        this.serverSideGridService.runSearchForGovernmentAgencyUsers(getRowsRequestDTO).subscribe((response: GridGetRowsResponseDTO) => {
    
        // REST Call finished successfully
        this.isValidQuery = response.isValidQuery;
    
        // Save the additional sort fields  (we will use when getting the next page)
        this.searchAfterClause = response.searchAfterClause;
    
        // Update total matches on the screen
        if (this.totalMatchesOnPageLoad == null){
        // Finished running the first search, so get the total number of records to display in the tab
        this.totalMatchesOnPageLoad = response.totalMatches;
        }
    
        this.displayedTotalMatches = this.generateFormattedTotalMatchesMessage(response.totalMatches);
    
        if (response.totalMatches == 0) {
        this.gridApi.showNoRowsOverlay();
        }
    
        // Load the data into the grid and turn on/off infinite scrolling
        // If lastRow == -1,       	then Infinite-Scrolling is turned ON
        // if lastRow == totalMatches, then infinite-scrolling is turned OFF
        if (!response.isValidQuery){
        //   User entered an invalid search
        params.successCallback([], 0)
        }
        else {
        // User entered a valid query
        params.successCallback(response.data, response.lastRow)
        }
        });
    
          }
      };
    
    private generateFormattedTotalMatchesMessage(aTotalMatches: number): string {
    if (aTotalMatches == 0) {
    return "";
    }

    if (aTotalMatches == 1) {
      return "1 Match";
    }

    // Use the toLocalString() to format 5000 --> "5,000"
    return String(aTotalMatches.toLocaleString()) + " Matches";
    }


    public downloadReport() {
    // Newer versions of ag grid do not have a builtin sort model so we must create one
    var sortState = this.gridColumnApi.getColumnState()
    .filter(function (s) {
    return s.sort != null;
    })
    .map(function (s) {
    return { colId: s.colId, sort: s.sort};
    });

    let exportDataDto: ExportDataDto = new ExportDataDto(this.gridApi.getFilterModel(), sortState, this.rawSearchQuery, NumericConstants.ALERT_SORTING_TYPE);

    this.exportService.downloadGovernmentAgencyUsersReportDataAsCsv(exportDataDto);
    }
    
    }

GridGetRowsRequestDTO is defined [here](./GridGetRowsRequestDTO.md)
