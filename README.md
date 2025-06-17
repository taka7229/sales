コントローラ
-----------------------------------------------------
/* ===== 参照名前空間 ===== */
using System;                                   // ◎  .NET 基本クラス
using System.Collections.Generic;               // ◎  コレクション系
using System.Linq;                              // ◎  LINQ クエリに必要
using System.Web.Mvc;                           // ◎  ASP.NET MVC の基本型
using FLMToursWebSystem.Models;                 // ◎  自作モデルを参照

namespace FLMToursWebSystem.Controllers          // ◎  コントローラ名前空間
{
    public class AggregateController : Controller // ◎  集計専用コントローラ。Controller を継承
    {
        private TourDBContext db = new TourDBContext(); // ◎  EF‐Code First の DB コンテキスト

        /* ─────────────────────────────────── */
        /*               メインメニュー                 */
        /* ─────────────────────────────────── */
        public ActionResult AggregateMenu() => View();  // ◎  /Aggregate/AggregateMenu を返すだけ

        /* ─────────────────────────────────── */
        /*               年 別 集 計                  */
        /* ─────────────────────────────────── */
        public ActionResult AggregateByYear()        // ◎  GET : 入力フォーム表示
        {
            ViewBag.CategoryList = MakeCategoryList(""); // ◎  ドロップダウン初期値
            ViewBag.ResultList   = null;                 // ◎  最初は結果なし
            return View();                               // ◎  /Views/Aggregate/AggregateByYear.cshtml
        }

        /* ---------- POST : 集計処理 ---------- */
        [HttpPost, ValidateAntiForgeryToken]            // ◎  CSRF 対策
        public ActionResult AggregateByYear(int? year,  // ◎  入力：年 (null 許容)
                                            string category) // ◎  入力：商品種別
        {
            bool hasError = false;                      // ◎  バリデーション判定用フラグ

            if (year == null)                           // ◎  年未入力
            {
                ViewBag.YearError = "年は入力必須です";
                hasError = true;
            }
            if (string.IsNullOrWhiteSpace(category))    // ◎  種別未入力
            {
                ViewBag.CategoryError = "商品種別は入力必須です";
                hasError = true;
            }

            if (hasError)                               // ◎  どちらか不足ならフォーム再表示
            {
                ViewBag.Year          = year;           // ◎  入力保持
                ViewBag.CategoryList  = MakeCategoryList(category);
                ViewBag.Info          = null;           // ◎  0件メッセージを消す
                return View();                          // ◎  Model は渡さない
            }

            /* ---------- 受注データ抽出 ---------- */
            var query =
                from o in db.Orders                    // ◎  Order テーブル
                where o.Date.Year == year              // ◎  年が一致
                   && (category == "ALL" ||            // ◎  ALL なら全件
                       o.Category == category)         // ◎  そうでなければ種別で絞る
                orderby o.InitTotal descending         // ◎  小計の大きい順
                select new AggregateByYearViewModel    // ◎  ビュー用 DTO を生成
                {
                    Date      = o.Date,
                    Item_Id   = o.Item_Id,
                    InitTotal = o.InitTotal            // ◎  小計は DB に既存
                };

            /* ---------- 0 件チェック ---------- */
            var list = query.ToList();                 // ◎  実行してメモリへ
            if (list.Count == 0)
            {
                ViewBag.Year         = year;
                ViewBag.Info         = "受注情報が存在しません";
                ViewBag.CategoryList = MakeCategoryList(category);
                return View();                         // ◎  Model なし
            }

            /* ---------- 合計金額計算 ---------- */
            int total = 0;
            foreach (var r in list) total += r.InitTotal;

            /* ---------- ViewBag へ結果 ---------- */
            ViewBag.Year         = year;
            ViewBag.CategoryStr  = category;
            ViewBag.Total        = total;
            ViewBag.CategoryList = MakeCategoryList(category);

            return View(list);                        // ◎  モデルとして結果リストを渡す
        }

        /* ─────────────────────────────────── */
        /*               月 別 集 計                  */
        /* ─────────────────────────────────── */
        public ActionResult AggregateByMonth()        // ◎  GET : フォーム
        {
            ViewBag.CategoryList = MakeCategoryList("");
            return View();
        }

        [HttpPost, ValidateAntiForgeryToken]          // ◎  POST : 集計
        public ActionResult AggregateByMonth(int? year, int? month, string category)
        {
            bool hasError = false;

            if (year  == null) { ViewBag.YearError  = "年は入力必須です";  hasError = true; }
            if (month == null) { ViewBag.MonthError = "月は入力必須です"; hasError = true; }
            if (string.IsNullOrWhiteSpace(category))
            {
                ViewBag.CategoryError = "商品種別は入力必須です";
                hasError = true;
            }

            if (hasError)                            // ◎  エラーなら入力を保持して戻る
            {
                ViewBag.Year         = year;
                ViewBag.Month        = month;
                ViewBag.CategoryList = MakeCategoryList(category);
                ViewBag.Info         = null;
                return View();
            }

            /* ---------- 検索 ---------- */
            var query =
                from o in db.Orders
                where o.Date.Year  == year
                   && o.Date.Month == month
                   && (category == "ALL" || o.Category == category)
                orderby o.InitTotal descending
                select new AggregateByMonthViewModel
                {
                    Date      = o.Date,
                    Item_Id   = o.Item_Id,
                    InitTotal = o.InitTotal
                };

            var list = query.ToList();

            if (list.Count == 0)                     // ◎  0件
            {
                ViewBag.Year         = year;
                ViewBag.Month        = month;
                ViewBag.Info         = "受注情報が存在しません";
                ViewBag.CategoryList = MakeCategoryList(category);
                return View();
            }

            int total = 0;
            foreach (var r in list) total += r.InitTotal;

            ViewBag.Year         = year;
            ViewBag.Month        = month;
            ViewBag.CategoryStr  = category;
            ViewBag.Total        = total;
            ViewBag.CategoryList = MakeCategoryList(category);

            return View(list);
        }

        /* ─────────────────────────────────── */
        /*             顧 客 別 集 計                 */
        /* ─────────────────────────────────── */
        public ActionResult AggregateByCustomer()     // ◎  GET : フォーム
        {
            return View();
        }

        [HttpPost, ValidateAntiForgeryToken]
        public ActionResult AggregateByCustomer(string customer_id)   // ◎  顧客コード
        {
            bool hasError = false;

            if (string.IsNullOrWhiteSpace(customer_id))
            {
                ViewBag.CustomerError = "顧客IDは入力必須です";
                hasError = true;
            }

            if (hasError)                            // ◎  未入力エラー
            {
                ViewBag.Customer = customer_id;      // ◎  入力保持
                return View();                       // ◎  Model なし
            }

            /* ---------- 抽出 ---------- */
            var query =
                from o in db.Orders
                where o.Customer == customer_id      // ◎  顧客一致
                orderby o.InitTotal descending
                select new AggregateByCustomerViewModel
                {
                    Date        = o.Date,
                    Customer_Id = o.Customer,
                    Item_Id     = o.Item_Id,
                    InitTotal   = o.InitTotal
                };

            var list = query.ToList();

            if (list.Count == 0)                     // ◎  0 件
            {
                ViewBag.Customer = customer_id;
                ViewBag.Info     = "受注情報が存在しません";
                return View();
            }

            int total = 0;
            foreach (var r in list) total += r.InitTotal;

            ViewBag.Customer = customer_id;
            ViewBag.Total    = total;

            return View(list);                       // ◎  結果を Model として渡す
        }

        /* ─────────────────────────────────── */
        /*         ドロップダウン作成ヘルパ              */
        /* ─────────────────────────────────── */
        private SelectList MakeCategoryList(string selected)
        {
            var src = new[]                      // ◎  無名型配列で選択肢を作成
            {
                new { Value = "",    Text = "選択してください" },
                new { Value = "ALL", Text = "ALL" },
                new { Value = "TUR", Text = "TUR (ツアー)" },
                new { Value = "FLT", Text = "FLT (航空券)" },
                new { Value = "HTL", Text = "HTL (ホテル)" }
            };
            return new SelectList(src, "Value", "Text", selected); // ◎  SelectList 生成
        }

        private SelectList MakeItemList(string selected)
        {
            var ids =
                (from o in db.Orders select o.Item_Id) // ◎  すべての Item_Id を取得
                .Distinct()                            // ◎  重複排除
                .OrderBy(id => id)                     // ◎  昇順
                .ToList();

            ids.Insert(0, "");                         // ◎  先頭に空文字を追加（"選択してください" 用）
            return new SelectList(ids, selected);      // ◎  SelectList 生成
        }
    }
}
