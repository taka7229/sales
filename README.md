menu
-------------------------------

@{
    /**/

    ViewBag.Title = "メインメニュー";
}


<div class="hero bg-light py-5">
    <div class="container">
        <h1>売り上げ集計システム</h1>

    </div>
</div>

<div class="container py-5">
    <div class="row">
        <div class="col-12 col-md-4">
            
            <p>@Html.ActionLink("年別受注集計", "AggregateByYear", "Aggregate", null, new { @class = "btn btn-outline-secondary" })</p>
        </div>

        <div class="col-12 col-md-4">
            
            <p>@Html.ActionLink("月別受注集計", "AggregateByMonth", "Aggregate", null, new { @class = "btn btn-outline-secondary" })</p>
        </div>

        <div class="col-12 col-md-4">
            
            <p>@Html.ActionLink("顧客別受注集計", "AggregateByCustomer", "Aggregate", null, new { @class = "btn btn-outline-secondary" })</p>
        </div>

    </div>
</div>

