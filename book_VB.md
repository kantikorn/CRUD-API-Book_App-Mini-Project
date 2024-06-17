```vbnet
Imports System.Net.Http
Imports System.Net.Http.Headers
Imports Newtonsoft.Json

Public Class book
    Inherits System.Web.UI.Page

    Private Sub Page_Load(ByVal sender As System.Object, ByVal e As System.EventArgs) Handles MyBase.Load
        If Not Page.IsPostBack Then
            LoadBooks()
        End If
    End Sub

    Private Sub LoadBooks()
        Using client As New HttpClient()
            client.BaseAddress = New Uri("http://localhost:5278/")
            client.DefaultRequestHeaders.Accept.Clear()
            client.DefaultRequestHeaders.Accept.Add(New MediaTypeWithQualityHeaderValue("application/json"))

            Dim response = client.GetAsync("api/Books").Result
            If response.IsSuccessStatusCode Then
                Dim books = JsonConvert.DeserializeObject(Of List(Of book))(response.Content.ReadAsStringAsync().Result)
                BooksGridView.DataSource = books
                BooksGridView.DataBind()
            End If
        End Using
    End Sub

    Protected Sub LoadBooksButton_Click(sender As Object, e As EventArgs)
        LoadBooks()
    End Sub

    Protected Sub AddBookButton_Click(sender As Object, e As EventArgs)
        Dim newBook As New book With {
            .BookName = BookNameTextBox.Text,
            .Price = Decimal.Parse(PriceTextBox.Text),
            .Category = CategoryTextBox.Text,
            .Author = AuthorTextBox.Text
        }

        Using client As New HttpClient()
            client.BaseAddress = New Uri("http://localhost:5278/")
            client.DefaultRequestHeaders.Accept.Clear()
            client.DefaultRequestHeaders.Accept.Add(New MediaTypeWithQualityHeaderValue("application/json"))

            Dim json As String = JsonConvert.SerializeObject(newBook, New JsonSerializerSettings With {
                .ReferenceLoopHandling = ReferenceLoopHandling.Ignore
            })
            Dim content As New StringContent(json, Encoding.UTF8, "application/json")
            Dim response = client.PostAsync("api/Books", content).Result
            response.EnsureSuccessStatusCode()
        End Using

        LoadBooks()
    End Sub

    Protected Sub BooksGridView_RowEditing(sender As Object, e As GridViewEditEventArgs)
        BooksGridView.EditIndex = e.NewEditIndex
        LoadBooks()
    End Sub

    Protected Sub BooksGridView_RowCancelingEdit(sender As Object, e As GridViewCancelEditEventArgs)
        BooksGridView.EditIndex = -1
        LoadBooks()
    End Sub

    Protected Sub BooksGridView_RowUpdating(sender As Object, e As GridViewUpdateEventArgs)
        Dim rowIndex As Integer = e.RowIndex
        Dim id As String = BooksGridView.DataKeys(rowIndex).Value.ToString()
        HiddenFieldRecordID.Value = id
        HiddenFieldOperationType.Value = "Update"

        ' Show the update confirmation modal
        ScriptManager.RegisterStartupScript(Me, Me.GetType(), "showUpdateModal", "showUpdateModal();", True)
    End Sub

    Protected Sub btnConfirmUpdate_Click(sender As Object, e As EventArgs)
        Dim id As String = HiddenFieldRecordID.Value
        Dim rowIndex As Integer = BooksGridView.EditIndex
        Dim row As GridViewRow = BooksGridView.Rows(rowIndex)

        Dim updatedBook As New book With {
            .Id = id,
            .BookName = DirectCast(row.Cells(1).Controls(0), TextBox).Text,
            .Price = Decimal.Parse(DirectCast(row.Cells(2).Controls(0), TextBox).Text),
            .Category = DirectCast(row.Cells(3).Controls(0), TextBox).Text,
            .Author = DirectCast(row.Cells(4).Controls(0), TextBox).Text
        }

        Using client As New HttpClient()
            client.BaseAddress = New Uri("http://localhost:5278/")
            client.DefaultRequestHeaders.Accept.Clear()
            client.DefaultRequestHeaders.Accept.Add(New MediaTypeWithQualityHeaderValue("application/json"))

            Dim json As String = JsonConvert.SerializeObject(updatedBook, New JsonSerializerSettings With {
                .ReferenceLoopHandling = ReferenceLoopHandling.Ignore
            })
            Dim content As New StringContent(json, Encoding.UTF8, "application/json")
            Dim response = client.PutAsync($"api/Books/{id}", content).Result
            response.EnsureSuccessStatusCode()
        End Using

        BooksGridView.EditIndex = -1
        LoadBooks()
    End Sub

    Protected Sub BooksGridView_RowDeleting(sender As Object, e As GridViewDeleteEventArgs)
        Dim rowIndex As Integer = e.RowIndex
        Dim id As String = BooksGridView.DataKeys(rowIndex).Value.ToString()
        HiddenFieldRecordID.Value = id
        HiddenFieldOperationType.Value = "Delete"

        ' Show the delete confirmation modal
        ScriptManager.RegisterStartupScript(Me, Me.GetType(), "showDeleteModal", "showDeleteModal();", True)
    End Sub

    Protected Sub btnConfirmDelete_Click(sender As Object, e As EventArgs)
        Dim id As String = HiddenFieldRecordID.Value

        Using client As New HttpClient()
            client.BaseAddress = New Uri("http://localhost:5278/")
            client.DefaultRequestHeaders.Accept.Clear()
            client.DefaultRequestHeaders.Accept.Add(New MediaTypeWithQualityHeaderValue("application/json"))

            Dim response = client.DeleteAsync($"api/Books/{id}").Result
            response.EnsureSuccessStatusCode()
        End Using

        LoadBooks()
    End Sub
    Public Class Book
        Public Property Id As String
        Public Property BookName As String
        Public Property Price As Decimal
        Public Property Category As String
        Public Property Author As String
    End Class
End Class
```

