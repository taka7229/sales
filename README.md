        /* ======================================= */
        /*             顧 客 別 集 計               */
        /* ======================================= */
        public ActionResult AggregateByCustomer()
        {
            //ViewBag.ItemList = MakeItemList("");
         
            return View();
        }

        [HttpPost, ValidateAntiForgeryToken]
        public ActionResult AggregateByCustomer(string customerName)
        {
            /* ── 入力必須チェック ───────────────── */
            //bool hasError = false;
            if (string.IsNullOrWhiteSpace(customerName))
            {
                ViewBag.CustomerError = "顧客名は入力必須です";
                //hasError = true;
                return View();
            }

            //if (hasError)
            //{
            //    /* 入力値を保持して再表示 */
            //    ViewBag.Customer = customerName;
            //    ViewBag.Info = null;          // 0 件メッセージは消す
            //    return View();                // Model は null
            //}

            /* 2-2 顧客テーブルから名前で検索（部分一致）*/
            var ids =
                (from c in db.Customers
                 where c.Isdelete == false                   // 論理削除除外
                    && c.Name.Contains(customerName)        // 部分一致
                 select c.Customer).ToList();               // ← 顧客ID のリスト

            if (ids.Count == 0)
            {
                ViewBag.Info = "該当する顧客が存在しません";
                ViewBag.CustomerName = customerName;        // 入力保持
                return View();
            }

            /* 2-3 Orders から抽出して集計 */
            var query =
                from o in db.Orders
                where ids.Contains(o.Customer)               // ← 取得した ID に含まれる
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
                ViewBag.CustomerName = customerName;
                ViewBag.Info = "受注情報が存在しません";
                return View();          // Model なし
            }

            /* ── 合計金額を計算 ─────────────────── */
            int total = 0;
            foreach (var r in list) total += r.InitTotal;

            ViewBag.Customer = customerName;
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
            return new SelectList(src, "Valu" +
                "e", "Text", selected);
        }

        
    }
}

--------------------------------------------------------------------------------
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
                       name="customerName"
                       value="@(ViewBag.CustomerName ?? "")"
                       class="form-control w-100" />
                <span class="text-danger">@ViewBag.CustomerNameError</span>
            </td>
        </tr>
    </table>
