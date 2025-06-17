controller
---------------------------------------------------------------
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
            //ViewBag.ItemList = MakeItemList("");
         
            return View();
        }

        [HttpPost, ValidateAntiForgeryToken]
        public ActionResult AggregateByCustomer(string customer_id)
        {
            /* ── 入力必須チェック ───────────────── */
            bool hasError = false;
            if (string.IsNullOrWhiteSpace(customer_id))
            {
                ViewBag.CustomerError = "顧客IDは入力必須です";
                hasError = true;
            }

            if (hasError)
            {
                /* 入力値を保持して再表示 */
                ViewBag.Customer = customer_id;
                ViewBag.Info = null;          // 0 件メッセージは消す
                return View();                // Model は null
            }

            /* ── データ抽出 ─────────────────────── */
            var query =
                from o in db.Orders
                where o.Customer == customer_id            // ← 顧客コードで抽出
                orderby o.InitTotal descending
                select new AggregateByCustomerViewModel
                {
                    Date = o.Date,
                    Customer_Id = o.Customer,
                    Item_Id = o.Item_Id,
                    InitTotal = o.InitTotal
                };

            var list = query.ToList();

            /* ── 0 件ならメッセージ ───────────────── */
            if (list.Count == 0)
            {
                ViewBag.Customer = customer_id;
                ViewBag.Info = "受注情報が存在しません";
                return View();          // Model なし
            }

            /* ── 合計金額を計算 ─────────────────── */
            int total = 0;
            foreach (var r in list) total += r.InitTotal;

            ViewBag.Customer = customer_id;
            ViewBag.Total = total;

            return View(list);          // Model に結果リストを渡す
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

customer
-------------------------------------------------------------------------------
@model IEnumerable<FLMToursWebSystem.Models.AggregateByCustomerViewModel>

@{
    ViewBag.Title = "顧客別受注集計";
}

<h4>顧客別受注集計</h4>

@* ───── 0 件メッセージなど ───── *@
@if (ViewBag.Info != null)
{
    <p class="text-danger fw-bold">@ViewBag.Info</p>
}

@* ───── 入力フォーム ───── *@
@using (Html.BeginForm("AggregateByCustomer",  // action
                       "Aggregate",            // controller
                       FormMethod.Post))
{
    @Html.AntiForgeryToken()

    <table>
        <tr>
            <td style="width:120px;">顧客ID</td>
            <td>
                <input type="text"
                       name="customer_id"
                       value="@(ViewBag.Customer ?? "")"
                       class="form-control w-100" />
                <span class="text-danger">@ViewBag.CustomerError</span>
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
                <th>顧客ID</th>
                <th>商品ID</th>
                <th style="width:120px;">小計</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var r in Model)
            {
                <tr>
                    <td>@r.Date.ToString("yyyy/MM/dd")</td>
                    <td>@r.Customer_Id</td>
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
