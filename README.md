```html
<%@ Page Language="vb" AutoEventWireup="false" CodeBehind="book.aspx.vb" Inherits="Book_app.book" %>

<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>Book Store</title>
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" rel="stylesheet" />
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
    <script type="text/javascript">
        function showUpdateModal() {
            $('#updateModal').modal('show');
        }

        function showDeleteModal() {
            $('#deleteModal').modal('show');
        }
    </script>
</head>
<body>
    <form id="form1" runat="server">
        <div>
            <h2>Book Store</h2>
            <asp:GridView ID="BooksGridView" runat="server" AutoGenerateColumns="False" DataKeyNames="Id"
                OnRowEditing="BooksGridView_RowEditing" OnRowCancelingEdit="BooksGridView_RowCancelingEdit"
                OnRowUpdating="BooksGridView_RowUpdating" OnRowDeleting="BooksGridView_RowDeleting">
                <Columns>
                    <asp:BoundField DataField="Id" HeaderText="ID" ReadOnly="True" />
                    <asp:BoundField DataField="BookName" HeaderText="Book Name" />
                    <asp:BoundField DataField="Price" HeaderText="Price" />
                    <asp:BoundField DataField="Category" HeaderText="Category" />
                    <asp:BoundField DataField="Author" HeaderText="Author" />
                    <asp:CommandField ShowEditButton="True" ShowDeleteButton="True" />
                </Columns>
            </asp:GridView>
            <br />
            <asp:Button ID="LoadBooksButton" runat="server" Text="Load Books" OnClick="LoadBooksButton_Click" />
            <br /><br />
            <asp:TextBox ID="BookNameTextBox" runat="server" Placeholder="Book Name"></asp:TextBox>
            <asp:TextBox ID="PriceTextBox" runat="server" Placeholder="Price"></asp:TextBox>
            <asp:TextBox ID="CategoryTextBox" runat="server" Placeholder="Category"></asp:TextBox>
            <asp:TextBox ID="AuthorTextBox" runat="server" Placeholder="Author"></asp:TextBox>
            <asp:Button ID="AddBookButton" runat="server" Text="Add Book" OnClick="AddBookButton_Click" />
        </div>

        <asp:HiddenField ID="HiddenFieldRecordID" runat="server" />
        <asp:HiddenField ID="HiddenFieldOperationType" runat="server" />

        <!-- Update Confirmation Modal -->
        <div id="updateModal" class="modal fade" role="dialog">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <button type="button" class="close" data-dismiss="modal"></button>
                        <h4 class="modal-title">Confirm Update</h4>
                    </div>
                    <div class="modal-body">
                        <p>Are you sure you want to update this record?</p>
                    </div>
                    <div class="modal-footer">
                        <asp:Button ID="btnConfirmUpdate" runat="server" Text="Update" CssClass="btn btn-primary" OnClick="btnConfirmUpdate_Click" />
                        <button type="button" class="btn btn-default" data-dismiss="modal">Cancel</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Delete Confirmation Modal -->
        <div id="deleteModal" class="modal fade" role="dialog">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <button type="button" class="close" data-dismiss="modal"></button>
                        <h4 class="modal-title">Confirm Delete</h4>
                    </div>
                    <div class="modal-body">
                        <p>Are you sure you want to delete this record?</p>
                    </div>
                    <div class="modal-footer">
                        <asp:Button ID="btnConfirmDelete" runat="server" Text="Delete" CssClass="btn btn-danger" OnClick="btnConfirmDelete_Click" />
                        <button type="button" class="btn btn-default" data-dismiss="modal">Cancel</button>
                    </div>
                </div>
            </div>
        </div>
    </form>
</body>
</html>
```
