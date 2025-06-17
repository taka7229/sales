AggregateController
-----------------------------------------------------------------------------
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web.Mvc;
using FLMToursWebSystem.Models;

namespace FLMToursWebSystem.Controllers
{
    public class AggregateController : Controller
    {
        private TourDBContext db = new TourDBContext();

        /* ───────── メニュー ───────── */
        public ActionResult AggregateMenu() => View();

        /* ======================================= */
        /*               年 別 集 計                */
        /* ======================================= */
        public ActionResult AggregateByYear()
        {
            ViewBag.CategoryList = MakeCategoryList("");
            ViewBag.ResultList = null;
            return View();
        }

        [HttpPost, ValidateAntiForgeryToken]
        public ActionResult AggregateByYear(int? year, string category)
        {
            /* ── 入力必須チェックを ViewBag で返す ── */
            bool hasError = false;

            if (year == null)
            {
                ViewBag.YearError = "年は入力必須です";
                hasError = true;
            }

            if (string.IsNullOrWhiteSpace(category))
            {
                ViewBag.CategoryError = "商品種別は入力必須です";
                hasError = true;
            }

            if (hasError)
            {
                /* 入力値を保持して再表示 */
                ViewBag.Year = year;
                ViewBag.CategoryList = MakeCategoryList(category);
                ViewBag.Info = null;              // 0 件メッセージはクリア
                return View();                            // Model は null のまま
            }


            var query =
                    from o in db.Orders
                    where o.Date.Year == year
                          && (category == "ALL" || o.Category == category)
                    orderby o.InitTotal descending
                    select new AggregateByYearViewModel
                    {
                        Date = o.Date,
                        Item_Id = o.Item_Id,
                        InitTotal = o.InitTotal
                    };

            /* ▼ 追加：０件チェック */
            var list = query.ToList();
            if (list.Count == 0)
            {
                ViewBag.Year = year;
                ViewBag.Info = "受注情報が存在しません";
                ViewBag.CategoryList = MakeCategoryList(category);
                
                return View();
            }

            int total = 0;
            foreach (var r in list) total += r.InitTotal;

            ViewBag.Year = year;
            ViewBag.CategoryStr = category;
            ViewBag.Total = total;
            ViewBag.CategoryList = MakeCategoryList(category);

            return View(list);
        }

        /* ======================================= */
        /*               月 別 集 計                */
        /* ======================================= */
        public ActionResult AggregateByMonth()
        {
            ViewBag.CategoryList = MakeCategoryList("");
            
            return View();
        }

        [HttpPost, ValidateAntiForgeryToken]
        public ActionResult AggregateByMonth(int? year, int? month, string category)
        {
            /* ── 入力必須チェック (ViewBag で返す) ── */
            bool hasError = false;

            if (year == null)
            {
                ViewBag.YearError = "年は入力必須です";
                hasError = true;
            }
            if (month == null)
            {
                ViewBag.MonthError = "月は入力必須です";
                hasError = true;
            }
            if (string.IsNullOrWhiteSpace(category))
            {
                ViewBag.CategoryError = "商品種別は入力必須です";
                hasError = true;
            }

            if (hasError)
            {
                ViewBag.Year = year;
                ViewBag.Month = month;
                ViewBag.CategoryList = MakeCategoryList(category);
                ViewBag.Info = null;            // 0 件メッセージは消す
                return View();                          // Model は null
            }

            var query =
                    from o in db.Orders
                    where o.Date.Year == year
                          && o.Date.Month == month
                          && (category == "ALL" || o.Category == category)
                    orderby o.InitTotal descending
                    select new AggregateByMonthViewModel
                    {
                        Date = o.Date,
                        Item_Id = o.Item_Id,
                        InitTotal = o.InitTotal
                    };

            /* ▼ 追加：０件チェック */
            var list = query.ToList();

            /* 0 件処理 */
            if (list.Count == 0)
            {
                ViewBag.Year = year;
                ViewBag.Month = month;
                ViewBag.Info = "受注情報が存在しません";
                ViewBag.CategoryList = MakeCategoryList(category);
                return View();                          // Model は null
            }

            int total = 0;
            foreach (var r in list) total += r.InitTotal;

            ViewBag.Year = year;
            ViewBag.Month = month;
            ViewBag.CategoryStr = category;
            ViewBag.Total = total;
            ViewBag.CategoryList = MakeCategoryList(category);

            return View(list);
        }

        /* ======================================= */
        /*             顧 客 別 集 計               */
        /* ======================================= */
        public ActionResult AggregateByCustomer()
        {
            ViewBag.ItemList = MakeItemList("");
         
            return View();
        }

        [HttpPost, ValidateAntiForgeryToken]
        public ActionResult AggregateByCustomer(string item_id)
        {
            ViewBag.ItemId = item_id;

            if (string.IsNullOrWhiteSpace(item_id))
            {
                ViewBag.ItemError = "商品IDを選択してください";
                ViewBag.ItemList = MakeItemList("");
                return View();
            }

            var query =
                from o in db.Orders
                where o.Item_Id == item_id

                select new AggregateByCustomerViewModel
                {
                    Date = o.Date,
                    Customer_Id = o.Customer,
                    Item_Id = o.Item_Id,
                    InitTotal = o.InitTotal
                };

            /* ▼ 追加：０件チェック */
            var list = query.ToList();
            if (list.Count == 0)
            {
                ViewBag.Info = "受注情報が存在しません";
                ViewBag.ItemList = MakeItemList(item_id);
                ViewBag.ResultList = null;
                return View();
            }

            int total = 0;
            foreach (var r in list) total += r.InitTotal;

            ViewBag.Total = total;
            ViewBag.ResultList = list;
            ViewBag.ItemList = MakeItemList(item_id);

            return View();
        }

        /* ───────── ドロップダウン作成 ───────── */
        private SelectList MakeCategoryList(string selected)
        {
            var src = new[]
            {
                new { Value = "",    Text = "選択してください" },
                new { Value = "ALL", Text = "ALL" },
                new { Value = "TUR", Text = "TUR (ツアー)" },
                new { Value = "FLT", Text = "FLT (航空券)" },
                new { Value = "HTL", Text = "HTL (ホテル)" }
            };
            return new SelectList(src, "Value", "Text", selected);
        }

        private SelectList MakeItemList(string selected)
        {
            var ids =
                (from o in db.Orders select o.Item_Id)
                .Distinct()
                .OrderBy(id => id)
                .ToList();

            ids.Insert(0, "");                       // 先頭に「選択してください」
            return new SelectList(ids, selected);
        }
    }
}


Order
------------------------------------------------------------------------
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using System.Linq;
using System.Web;

namespace FLMToursWebSystem.Models
{
    [Table("Order")]
    public class Order
    {
        
        [Key]
        
        public int OrderNo { get; set; }        
        public int Num { get; set; }
        [DataType(DataType.Date)]
        public DateTime Date { get; set; }

       
        public string Customer { get; set; }

       
        public string Item_Id { get; set; }

      
        public string Category { get; set; }

       
        public int Price { get; set; }

       
        public int Quantity { get; set; }

       
        public int InitTotal { get; set; }

    }
}

aggregateByCustomerViewModel
------------------------------------------------------
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Web;

namespace FLMToursWebSystem.Models
{
    public class AggregateByCustomerViewModel
    {
        public DateTime Date { get; set; }

        [Required(ErrorMessage = "顧客IDを入力してください")]
        public string Customer_Id { get; set; }
        public string Item_Id { get; set; }
        public int InitTotal { get; set; }

    }
}

aggregateByMonthViewModel
-------------------------------------------------------------------
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;

namespace FLMToursWebSystem.Models
{
    public class AggregateByMonthViewModel
    {
        public DateTime Date { get; set; }
        public string Item_Id { get; set; }
        public int InitTotal { get; set; }

    }
}

aggregateByYearViewModel
---------------------------------------------------------------------
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;

namespace FLMToursWebSystem.Models
{
    public class AggregateByYearViewModel
    {
        public DateTime Date { get; set; }
        public string Item_Id { get; set; }
        public int InitTotal { get; set; }

    }
}

aggregateByMonth
-----------------------------------------------------------------------------------
@model IEnumerable<FLMToursWebSystem.Models.AggregateByMonthViewModel>

@{
    ViewBag.Title = "月別受注集計";
}

<h4>月別受注集計</h4>

@* ───── 0 件メッセージなど ───── *@
@if (ViewBag.Info != null)
{
    @ViewBag.Info
}

@* ───── 入力フォーム ───── *@
@using (Html.BeginForm("AggregateByMonth",            // action
                       "Aggregate",                   // controller
                       FormMethod.Post))
{
    @Html.AntiForgeryToken()

    <table>
        <tr>
            <td style="width:120px;">年</td>
            <td>
                <input type="number"
                       name="year"
                       value="@(ViewBag.Year ?? "")"
                       class="form-control w-100" />
                       @ViewBag.YearError
            </td>
        </tr>
        <tr>
            <td>月</td>
            <td>
                <input type="number"
                       name="month"
                       value="@(ViewBag.Month ?? "")"
                       class="form-control w-100"
                       min="1" max="12" />
                       @ViewBag.MonthError
            </td>
        </tr>
        <tr>
            <td>商品種別</td>
            <td>
                @Html.DropDownList("category",
                                   (SelectList)ViewBag.CategoryList,
                                   new { @class = "form-control w-100" })
                                   @ViewBag.CategoryError
            </td>
        </tr>
    </table>

    <button type="submit" class="btn btn-primary mt-3">集計</button>
}

@* ───── 結果テーブル ───── *@
@if (Model != null && Model.Any())
{
    <hr />
    <p class="text-danger fw-bold">
        受注金額合計：@string.Format("{0:N0}", ViewBag.Total) 円
    </p>

    <table class="table table-bordered table-sm">
        <thead class="text-center">
            <tr>
                <th>受注日</th>
                <th>商品ID</th>
                <th style="width:120px;">小計</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var r in Model)
            {
                <tr>
                    <td>@r.Date.ToString("yyyy/MM/dd")</td>
                    <td>@r.Item_Id</td>
                    <td class="text-end">@r.InitTotal.ToString("N0")</td>
                </tr>
            }
        </tbody>
    </table>
}

@Html.ActionLink("メニューへ戻る",
                 "AggregateMenu",
                 "Aggregate",
                 null,
                 new { @class = "btn btn-link" })

@section Scripts{
    @Scripts.Render("~/bundles/jqueryval")
}

aggregateByYear
-----------------------------------------------------------------------
@model IEnumerable<FLMToursWebSystem.Models.AggregateByYearViewModel>

@{
    ViewBag.Title = "年別受注集計";
}

<h4>年別受注集計</h4>


@* ───── 情報メッセージ (0 件など) ───── *@
@if (ViewBag.Info != null)
{
    @ViewBag.Info
}

@* ───── 入力フォーム ───── *@
@using (Html.BeginForm("AggregateByYear", "Aggregate", FormMethod.Post))
{
    @Html.AntiForgeryToken()

    <table>
        <tr>
            <td style="width:120px;">年</td>
            <td>
                <input type="number" name="year"
                       value="@(ViewBag.Year ?? "")"
                       class="form-control w-100" />
                @ViewBag.YearError
            </td>
        </tr>
        <tr>
            <td>商品種別</td>
            <td>
                @Html.DropDownList("category",
                                   (SelectList)ViewBag.CategoryList,
                                   new { @class = "form-control w-100" })
                @ViewBag.CategoryError
            </td>
        </tr>
    </table>

    <button type="submit" class="btn btn-primary mt-3">集計</button>
}


@* ───── 結果テーブル ───── *@
@if (Model != null && Model.Any())
{
    <hr />
    <p class="text-danger fw-bold">
        受注金額合計：@string.Format("{0:N0}", ViewBag.Total) 円
    </p>

    <table class="table table-bordered table-sm">
        <thead class="text-center">
            <tr>
                <th>受注日</th>
                <th>商品ID</th>
                <th style="width:120px;">小計</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var r in Model)
            {
                <tr>
                    <td>@r.Date.ToString("yyyy/MM/dd")</td>
                    <td>@r.Item_Id</td>
                    <td class="text-end">@r.InitTotal.ToString("N0")</td>
                </tr>
            }
        </tbody>
    </table>
}

@Html.ActionLink("メニューへ戻る", "AggregateMenu", "Aggregate", null, new { @class = "btn btn-link" })

@section Scripts{
    @Scripts.Render("~/bundles/jqueryval")
}

aggregateMenu
----------------------------------------------------------------------------------------
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



