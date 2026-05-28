![img_1.png](img_1.png)

This page populates the ag-grid with data from Elasticsearch and each of the columns is filterable and sortable (based on both the ag-grid configuration AND the ES mapping).  The search function 
will search for matches across the fields and will also turn red if the search term is invalid (e.g. searching for "this AND that NOT this").  The grid page also contains code in the "Settings" button
to reset and resize the grid.  There is also the ability to download the report.  
Lastly, the page shows how many users are returned (in the top left) but ALSO how many matches are returned for your search - in this image there is no active search so the number of matches is equal
to the number of returned users

    <div class="default-page-layout">

      <!-- Header Bar Div Column -->
      <div class="default-page-header">
        <span class="font-normal text-2xl ml-5 uppercase" title="{{Constants.APPLICATION_NAME}} - Users Report">{{Constants.APPLICATION_NAME}} - Users Report</span>
      </div>
    
      <!-- Page Content Wrapper -->
      <div class="flex flex-col h-full w-full bg-defaultPageColor pr-5 pl-5 pb-5">

    <!-- Page Content Div Column -->
    <div class="flex flex-col w-full mt-5 h-full rounded-t">

      <!-- Tab & Search Row -->
      <div class="flex flex-row w-full h-[64px] relative flex-shrink-0">


        <!-- Tab -->
        <div class="flex flex-row items-center absolute bg-white rounded-t px-3 py-2 border-x border-t border-borderColor h-full w-[150px] top-[1px]">
          <div class="w-[5px] h-full float-left bg-[#1E3059] rounded mr-2.5 flex-shrink-0"></div>
          <!-- Note: Negative margin here because of line height on the numbers -->
          <div class="flex flex-col pt-2">
            <div class="h-[30px] w-[150px] flex place-content-start">
              <!-- Title (count) -->
              <ng-container>
                <span class="text-2xl font-extrabold">{{this.totalMatchesOnPageLoad}}</span>
              </ng-container>
            </div>

            <div class="h-[30px] flex place-content-start">
              <!-- Sub Title (context) -->
              <span>Users</span>
            </div>
          </div>
        </div>


        <!-- Searchbar Container -->
        <div class="h-full w-full py-2 flex flex-row pl-[158px]">
          <!-- Searchbar -->
          <div class="w-full rounded border-borderColor border justify-center flex flex-row gap-2.5 pl-3.5 overflow-hidden"
               [ngClass]="{
                      'searchBoxValid':    this.isValidQuery,
                      'searchBoxInvalid':  !this.isValidQuery
                       }">


            <!-- Search box -->
            <input matInput type="text" #searchBox
                   class="w-full outline-none"
                   placeholder="Search..."
                   autocomplete="off"
                   title="Search Box"
                   [ngClass]="{
                      'searchBoxValid':    this.isValidQuery,
                      'searchBoxInvalid':  !this.isValidQuery
                       }"
                   [(ngModel)]="this.rawSearchQuery" (keyup.enter)="this.runSearch()"
                   aria-label="Search Box"/>

            <!-- Clear Icon -->
            <span class="flex clickable items-center justify-center"
                  (click)="this.clearSearch(); searchBox.value=''" aria-label="Clear Search" title="Clear Search">
              <i class="fa-solid fa-xmark-large"></i>
            </span>

            <!-- Search Icon -->
            <div class="bg-blue-950 rounded-r w-[42px] items-center justify-center clickable text-white flex h-full"
                 aria-label="Search" title="Search" (click)="this.runSearch()">
              <i class="fa-regular fa-search"></i>
            </div>
          </div>

        </div>

      </div>

      <!-- Searchbar Column -->
      <div class="flex flex-row w-full bg-white rounded-tr h-10 flex-shrink-0 items-center border-x border-t border-borderColor pr-3">

        <div class="flex flex-row basis-[200px] place-content-start items-center pt-1 pb-2">
          <!-- L E F T      S I D E     O F     R O W  -->

          <!-- Grid Options Popup Menu -->
          <button rbr-stripped-button no-hover class="-ml-1"
                  [matMenuTriggerFor]="myMenu"
                  type="button" title="Settings" aria-label="Settings">
            <i class="fa-xl fa-solid fa-sliders"></i>
            <span class="font-extrabold">Settings</span>
          </button>

          <mat-menu #myMenu="matMenu">
            <button mat-menu-item title="Reset Grid" aria-label="Reset Grid" (click)="this.resetGrid()">Reset Grid
            </button>
            <button mat-menu-item title="Autosize Grid" aria-label="Autosize Grid" (click)="this.autoSizeGrid()">
              Autosize Grid
            </button>
          </mat-menu>

          <button (click)="this.downloadReport()" rbr-stripped-button no-hover
                  type="button" title="Download Users Report" aria-label="Download Users Report">
            <div class="flex flex-row gap-1 items-center">
              <i class="fa fa-download"></i>
              <span class="font-extrabold">Download Report</span>
            </div>
          </button>

        </div>

        <div class="flex flex-grow place-content-center items-center py-2">
          <!-- C E N T E R      S I D E     O F     R O W  -->
        </div>
        <div  class="flex ml-auto">
          <!-- R I G H T      S I D E     O F     R O W  -->

          <!-- Show the Total Number of Matches -->
          <div class="flex flex-row gap-5 h-full items-center">
            <span class="italic text-primary font-extrabold h-full pb-0.5">{{ this.displayedTotalMatches }}</span>
            <app-grid-preference-check [saveGridColumnStateEventsSubject]="this.saveGridColumnStateEventsSubject"></app-grid-preference-check>
          </div>

        </div>
      </div>

      <!-- Grid Container Column -->
      <div class="bg-white w-full h-full border-x border-b rounded-b overflow-hidden border-borderColor">
        <div class="w-full h-full">
          <ag-grid-angular
            style="width: 100%; height: 100%;"
            class="ag-theme-balham"
            [class.single-select]="gridOptions.rowSelection === 'single'"
            [gridOptions]="this.gridOptions"
            [columnDefs]="this.columnDefs"
            [defaultColDef]="this.defaultColDefs"
            (firstDataRendered)="this.firstDataRendered()"
            (gridReady)="this.onGridReady($event)">
          </ag-grid-angular>
        </div>
      </div>
    </div>
     </div>`
