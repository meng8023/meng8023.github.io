---
title: GridLookUpEdit多列模糊查询
tags: Dev GridLookUpEdit 多列模糊查询  
key: 20190821
---

内容标题
===============


属性设置
-------
{% highlight ruby linenos %}

		this.repositoryItemGridLookUpEdit.AutoComplete = false;
		this.repositoryItemGridLookUpEdit.AutoHeight = false;
		this.repositoryItemGridLookUpEdit.ImmediatePopup = true;
		this.repositoryItemGridLookUpEdit.NullText = "";
		this.repositoryItemGridLookUpEdit.PopupFilterMode = DevExpress.XtraEditors.PopupFilterMode.Contains;
		this.repositoryItemGridLookUpEdit.TextEditStyle = DevExpress.XtraEditors.Controls.TextEditStyles.Standard;

{% endhighlight ruby %}

扩展代码
----------
Dev 提供的 GridLookUpEdit 不能进行多列模糊查询，所以需要自己扩展一下方法
注册一个EditValueChanging事件来满足我们的需求
注意其中的  repositoryItemGridLookUpEdit.PopupView.FindFilterText = string.Empty;
如果没有清除过滤文本，会出现第二次检索时，只能在第一次检索的数据基础上进行
注意：repositoryItemGridLookUpEdit 控件中可能不会出现这个问题，GridLookUpEdit 控件中存在这个问题
{% highlight ruby linenos %}
	 private void RepositoryItemGridLookUpEdit_EditValueChanging(object sender, DevExpress.XtraEditors.Controls.ChangingEventArgs e)
	{
		 repositoryItemGridLookUpEdit.PopupView.FindFilterText = string.Empty;
		this.BeginInvoke(new System.Windows.Forms.MethodInvoker(() =>
		{
			GridLookUpEdit edit = sender as GridLookUpEdit;
			DevExpress.XtraGrid.Views.Grid.GridView view = edit.Properties.View as DevExpress.XtraGrid.Views.Grid.GridView;
			//获取GriView私有变量
			System.Reflection.FieldInfo extraFilter = view.GetType().GetField("extraFilter", System.Reflection.BindingFlags.NonPublic | System.Reflection.BindingFlags.Instance);
			List<DevExpress.Data.Filtering.CriteriaOperator> columnsOperators = new List<DevExpress.Data.Filtering.CriteriaOperator>();
			foreach (GridColumn col in view.VisibleColumns)
			{
				if (col.Visible && col.ColumnType == typeof(string))
					columnsOperators.Add(new DevExpress.Data.Filtering.FunctionOperator(DevExpress.Data.Filtering.FunctionOperatorType.Contains,
				new DevExpress.Data.Filtering.OperandProperty(col.FieldName),
				new DevExpress.Data.Filtering.OperandValue(edit.Text)));
			}
			string filterCondition = new DevExpress.Data.Filtering.GroupOperator(DevExpress.Data.Filtering.GroupOperatorType.Or, columnsOperators).ToString();
			extraFilter.SetValue(view, filterCondition);
			//获取GriView中处理列过滤的私有方法
			System.Reflection.MethodInfo ApplyColumnsFilterEx = view.GetType().GetMethod("ApplyColumnsFilterEx", System.Reflection.BindingFlags.NonPublic | System.Reflection.BindingFlags.Instance);
			ApplyColumnsFilterEx.Invoke(view, null);
		}));

	}
{% endhighlight ruby %}