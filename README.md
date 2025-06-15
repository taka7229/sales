# sales

HotelController.cs
---------------------------------------------------------
using System;
using System.Collections.Generic;
using System.Data;
using System.Data.Entity;
using System.Linq;
using System.Net;
using System.Web;
using System.Web.Mvc;
using PagedList;
using SalesSystem2.Models;

namespace SalesSystem2.Controllers
{
    public class HotelController : Controller
    {
        private SalesDbContext db = new SalesDbContext();

        
        public const int PageSize = 5;
        // GET: Hotels
        public ActionResult HotelList(int? page)
        {
            int pageNumber = page ?? 1;

            // Isdelete == false のホテルだけを取り出して一覧へ
            var hotels = (from h in db.Hotels
                          where !h.Isdelete
                          orderby h.HotelId// ← 論理削除されていない
                          select h).ToList();      // 一覧用に ToList() で確定

            return View(hotels.ToPagedList(pageNumber,PageSize));
        }

        // GET: Hotels/Create
        public ActionResult HotelRegist()
        {
            return View();
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult HotelRegist(Hotel hotel)
        {
            db.Hotels.Add(hotel);
            db.SaveChanges();
            ViewBag.Info = "登録が完了しました。";
            return View();

        }

        // GET: Hotels/Edit/5
        public ActionResult HotelEdit(string id)
        {
            var hotel = (from h in db.Hotels
                         where h.HotelId == id
                               && !h.Isdelete      // ← 論理削除を除外
                         select h).FirstOrDefault();
            return View(hotel);
        }

        // POST: Hotels/Edit/5
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult HotelEdit(Hotel hotel)
        {
            db.Entry(hotel).State = EntityState.Modified;
            db.SaveChanges();
            ViewBag.Info = "変更が完了しました。";

            return View(hotel);
        }

        public ActionResult HotelDelete(string id)
        {
            //FirstOrDefalutでコレクションから分割できるようにする
            var hotel = (from h in db.Hotels
                         where h.HotelId == id
                                   
                         select h).FirstOrDefault();
           
            return View(hotel);
        }

        // POST: Hotels/Delete/5
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult HotelDelete(Hotel hotel)
        {

            hotel.Isdelete = true;

            //更新であることを明示
            db.Entry(hotel).State = EntityState.Modified;
            db.SaveChanges();

            ViewBag.Info = "削除が完了しました";

            return View(hotel);
        }
    }
}

Hotel.cs
-----------------------------------------------------------------------------------------------------
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using System.Linq;
using System.Web;

namespace SalesSystem2.Models
{
    [Table("Hotel")]
    public class Hotel
    {
        // ホテルID（HTL+数字6桁）
        [Key]
        [Required(ErrorMessage = "ホテルIDを入力してください")]
        [RegularExpression(@"HTL\d{6}", ErrorMessage = "「HTL」＋数字6桁で入力してください")]
        [Display(Name = "ホテルID")]
        public string HotelId { get; set; }

        // ホテル名
        [Required(ErrorMessage = "ホテル名を入力してください")]
        [Display(Name = "ホテル名")]
        public string HotelName { get; set; }

        // 都市ID（数字2桁）
        [Required(ErrorMessage = "都市IDを入力してください")]
        [RegularExpression(@"\d{2}", ErrorMessage = "数字2桁で入力してください")]
        [Display(Name = "都市ID")]
        public int City_Id { get; set; }

        // 宿泊日
        [Required(ErrorMessage = "宿泊日を入力してください")]
        [DataType(DataType.Date)]
        [Display(Name = "宿泊日")]
        public DateTime Date { get; set; }

        // 基本価格
        [Required(ErrorMessage = "基本価格を入力してください")]
        [Range(0, int.MaxValue, ErrorMessage = "0以上の整数で入力してください")]
        [Display(Name = "基本価格")]
        public int Price { get; set; }

        // 在庫数
        [Required(ErrorMessage = "在庫数を入力してください")]
        [Range(0, int.MaxValue, ErrorMessage = "0以上の整数で入力してください")]
        [Display(Name = "在庫数")]
        public int Stock { get; set; }

        // 論理削除フラグ
        public bool Isdelete { get; set; }
    }
}

SalesDbContext.cs
--------------------------------------------------------------------------------------------
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using SalesSystem2.Models;
using System.Data.Entity;


namespace SalesSystem2.Models
{
    public class SalesDbContext :DbContext
    {
        public DbSet<Hotel> Hotels { get; set; }
    }
}

HotelDelete.cshtml
------------------------------------------------------------------------------------------------------

@model SalesSystem2.Models.Hotel

@{
    ViewBag.Title = "得意先削除";
}

<!-- ★ ① 追加：読み取り専用 input を薄い灰色に -->
<style>
    .form-control[readonly] {
        background-color: #f0f0f0; /* 薄い灰色 */
        color: #495057; /* 文字色は既定と同じ */
    }
</style>


<div class="hero bg-light py-5">
    <div class="container">
        <h2>得意先削除</h2>
        @ViewBag.Info

    </div>
</div>

<h4 class="mt-3">削除</h4>

@using (Html.BeginForm("HotelDelete", "Hotel", new { id = Model.HotelId }, FormMethod.Post))
{
    @Html.AntiForgeryToken()


    <div class="container mt-5">
        <hr />

        <!-- ホテルID -->
        <div class="row mb-3">
            @Html.LabelFor(m => m.HotelId, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.TextBoxFor(m => m.HotelId, new { @class = "form-control", @readonly = "readonly" })
            </div>
        </div>

        <!-- ホテル名 -->
        <div class="row mb-3">
            @Html.LabelFor(m => m.HotelName, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.TextBoxFor(m => m.HotelName, new { @class = "form-control", @readonly = "readonly" })
            </div>
        </div>

        <!-- 都市ID -->
        <div class="row mb-3">
            @Html.LabelFor(m => m.City_Id, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.TextBoxFor(m => m.City_Id, new { @class = "form-control", @readonly = "readonly" })
            </div>
        </div>

        <!-- 日付 -->
        <div class="row mb-3">
            @Html.LabelFor(m => m.Date, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.TextBox("Date", Model.Date.ToString("yyyy/MM/dd"),
                              new { @class = "form-control", @readonly = "readonly" })
            </div>
        </div>

        <!-- 基本価格 -->
        <div class="row mb-3">
            @Html.LabelFor(m => m.Price, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.TextBoxFor(m => m.Price, new { @class = "form-control", @readonly = "readonly" })
            </div>
        </div>

        <!-- 在庫数 -->
        <div class="row mb-3">
            @Html.LabelFor(m => m.Stock, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.TextBoxFor(m => m.Stock, new { @class = "form-control", @readonly = "readonly" })
            </div>
        </div>
    </div>



    if (ViewBag.Info == null) { 
    <div class="row mb-3">
        <div class="col-md-10 offset-md-2">
            <input type="submit" class="btn btn-primary" value="削除確定" />
        </div>
    </div>
     }

 }


<div class="d-flex gap-2">
    @Html.ActionLink("一覧へ", "HotelList", "Hotel", null, null)

</div>

@section Scripts{
    @Scripts.Render("~/bundles/jqueryval")
}

HotelEdit.cshtml
---------------------------------------------------------------------------------------------------------

@model SalesSystem2.Models.Hotel

@{
    ViewBag.Title = "得意先編集";
}

<!-- ★ ① 追加：読み取り専用 input を薄い灰色に -->
<style>
    .form-control[readonly] {
        background-color: #f0f0f0; /* 薄い灰色 */
        color: #495057; /* 文字色は既定と同じ */
    }
</style>

<div class="hero bg-light py-5">
    <div class="container">
        <h1>得意先編集</h1>
        @ViewBag.Info

    </div>
</div>

@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()

    <div class="container mt-5">

        <hr />

        <!-- ホテルID（読み取り専用） -->
        <div class="row mb-3">
            @Html.LabelFor(m => m.HotelId, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.TextBoxFor(m => m.HotelId, new { @class = "form-control", @readonly = "readonly" })   <!-- ★ -->
                @Html.ValidationMessageFor(m => m.HotelId, "", new { @class = "text-danger" })
            </div>
        </div>
        <div class="row mb-3">
            @Html.LabelFor(model => model.HotelName, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.HotelName, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.HotelName, "", new { @class = "text-danger" })
            </div>
        </div>
        <div class="row mb-3">
            @Html.LabelFor(model => model.City_Id, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.City_Id, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.City_Id, "", new { @class = "text-danger" })
            </div>
        </div>
        <div class="row mb-3">
            @Html.LabelFor(model => model.Date, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.Date, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.Date, "", new { @class = "text-danger" })
            </div>
        </div>

        <!-- 基本価格（読み取り専用） -->
        <div class="row mb-3">
            @Html.LabelFor(m => m.Price, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.TextBoxFor(m => m.Price, new { @class = "form-control", @readonly = "readonly" })   <!-- ★ -->
                @Html.ValidationMessageFor(m => m.Price, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="row mb-3">
            @Html.LabelFor(model => model.Stock, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.Stock, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.Stock, "", new { @class = "text-danger" })
            </div>
        </div>

        @if (ViewBag.Info == null)
        {
        <div class="row mb-3">
            <div class="col-md-10 offset-md-2">
                <input type="submit" class="btn btn-primary" value="編集確定" />

            </div>
        </div>
        }
    </div>
}


<div class="d-flex gap-2">
    @Html.ActionLink("一覧へ", "HotelList", "Hotel", null, null)

</div>

@section Scripts{
    @Scripts.Render("~/bundles/jqueryval")
}

HotelList.cshtml
---------------------------------------------------------------------------------------------------------
@model IEnumerable<SalesSystem2.Models.Hotel>
@using PagedList.Mvc
@using PagedList

@{
    ViewBag.Title = "Index";
}

<style>
    /* 画面全体の背景をパネルと同じパステルグリーンに */
    body {
        background: #d7e8c9;
    }

    /* パネル内テーブルのセルも同じ色に */
    .panel-green table, /* 表自体 */
    .panel-green th, /* 見出しセル */
    .panel-green td { /* データセル */
        background: #d7e8c9;
        border: 1px solid #000; /* ← 黒 (#000) */
    }
</style>

<div class="container panel-green p-4 my-3">
    <h2>ホテル管理システム</h2>
    <h4>ホテル一覧</h4>



    @Html.ActionLink("登録", "HotelRegist")

    <table class="table table-bordered table-sm bg-white">
        <thead class="table-secondary text-center">
            <tr>
                <th>
                    @Html.DisplayNameFor(model => model.HotelId)
                </th>
                <th>
                    @Html.DisplayNameFor(model => model.HotelName)
                </th>
                <th>
                    @Html.DisplayNameFor(model => model.City_Id)
                </th>
                <th>
                    @Html.DisplayNameFor(model => model.Date)
                </th>
                <th>
                    @Html.DisplayNameFor(model => model.Price)
                </th>
                <th>
                    @Html.DisplayNameFor(model => model.Stock)
                </th>

                <th></th>
                <th></th>
            </tr>

            @foreach (var item in Model)
            {
                <tr>
                    <td>
                        @Html.DisplayFor(modelItem => item.HotelId)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.HotelName)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.City_Id)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.Date)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.Price)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.Stock)
                    </td>

                    <td>
                        @Html.ActionLink("変更", "HotelEdit", "Hotel", new { id = item.HotelId }, null)
                    </td>
                    <td>
                        @Html.ActionLink("削除", "HotelDelete", "Hotel", new { id = item.HotelId }, null)
                    </td>
                </tr>
            }

    </table>

</div>

HotelRegist.cshtml
-------------------------------------------------------------------------------------------------------------------------
@model SalesSystem2.Models.Hotel

@{
    ViewBag.Title = "ホテル登録";
}


<div class="hero bg-light py-5">
    <div class="container">
        <h1>ホテル登録</h1>
        @ViewBag.Info

    </div>
</div>

@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()

    <div class="container mt-5">
        <h4>商品購入</h4>
        <hr />
        <div class="row mb-3">
            @Html.LabelFor(model => model.HotelId, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.HotelId, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.HotelId, "", new { @class = "text-danger" })
            </div>
        </div>
        <div class="row mb-3">
            @Html.LabelFor(model => model.HotelName, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.HotelName, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.HotelName, "", new { @class = "text-danger" })
            </div>
        </div>
        <div class="row mb-3">
            @Html.LabelFor(model => model.City_Id, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.City_Id, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.City_Id, "", new { @class = "text-danger" })
            </div>
        </div>
        <div class="row mb-3">
            @Html.LabelFor(model => model.Date, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.Date, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.Date, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="row mb-3">
            @Html.LabelFor(model => model.Price, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.Price, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.Price, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="row mb-3">
            @Html.LabelFor(model => model.Stock, new { @class = "col-form-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.Stock, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.Stock, "", new { @class = "text-danger" })
            </div>
        </div>

        @if (ViewBag.Info == null)
        {
        <div class="row mb-3">
            <div class="col-md-10 offset-md-2">
                <input type="submit" class="btn btn-primary" value="登録" />

            </div>
        </div>
        }
    </div>
}


<div class="d-flex gap-2">
    @Html.ActionLink("一覧へ", "HotelList", "Hotel", null, null)

</div>

@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
}
