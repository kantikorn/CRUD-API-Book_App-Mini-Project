```vbnet
Imports System.Net.Http
Imports System.Net.Http.Headers
Imports Newtonsoft.Json

Public Class book
    Inherits System.Web.UI.Page

    ' ฟังก์ชันที่จะถูกเรียกเมื่อหน้าถูกโหลดครั้งแรก
    Private Sub Page_Load(ByVal sender As System.Object, ByVal e As System.EventArgs) Handles MyBase.Load
        If Not Page.IsPostBack Then
            ' เรียกฟังก์ชัน LoadBooks เพื่อโหลดข้อมูลหนังสือมาแสดงใน GridView
            LoadBooks()
        End If
    End Sub

    ' ฟังก์ชันสำหรับโหลดข้อมูลหนังสือจาก API
    Private Sub LoadBooks()
        Using client As New HttpClient()
            ' กำหนดฐานที่อยู่ของ API
            client.BaseAddress = New Uri("http://localhost:5278/")
            client.DefaultRequestHeaders.Accept.Clear()
            client.DefaultRequestHeaders.Accept.Add(New MediaTypeWithQualityHeaderValue("application/json"))

            ' ส่งคำร้องขอ GET ไปยัง API เพื่อดึงข้อมูลหนังสือ
            Dim response = client.GetAsync("api/Books").Result
            If response.IsSuccessStatusCode Then
                ' ถ้าคำร้องขอสำเร็จ ให้ Deserialize ข้อมูล JSON เป็นรายการหนังสือ
                Dim books = JsonConvert.DeserializeObject(Of List(Of Book))(response.Content.ReadAsStringAsync().Result)
                ' กำหนด datasource ของ GridView เป็นรายการหนังสือ
                BooksGridView.DataSource = books
                ' สั่งให้ GridView แสดงผล
                BooksGridView.DataBind()
            End If
        End Using
    End Sub

    ' ฟังก์ชันที่จะถูกเรียกเมื่อปุ่ม LoadBooksButton ถูกคลิก
    Protected Sub LoadBooksButton_Click(sender As Object, e As EventArgs)
        LoadBooks()
    End Sub

    ' ฟังก์ชันที่จะถูกเรียกเมื่อปุ่ม AddBookButton ถูกคลิก
    Protected Sub AddBookButton_Click(sender As Object, e As EventArgs)
        ' กำหนดตัวแปรเพื่อเก็บค่าจาก TextBox ที่รับค่า
        Dim newBook As New Book With {
            .BookName = BookNameTextBox.Text,
            .Price = Decimal.Parse(PriceTextBox.Text),
            .Category = CategoryTextBox.Text,
            .Author = AuthorTextBox.Text
        }

        Using client As New HttpClient()
            ' กำหนดฐานที่อยู่ของ API
            client.BaseAddress = New Uri("http://localhost:5278/")
            client.DefaultRequestHeaders.Accept.Clear()
            client.DefaultRequestHeaders.Accept.Add(New MediaTypeWithQualityHeaderValue("application/json"))

            ' สร้างหนังสือใหม่ในรูปแบบ JSON
            Dim json As String = JsonConvert.SerializeObject(newBook, New JsonSerializerSettings With {
                .ReferenceLoopHandling = ReferenceLoopHandling.Ignore
            })
            Dim content As New StringContent(json, Encoding.UTF8, "application/json")
            ' ส่งคำร้องขอ POST ไปยัง API เพื่อเพิ่มหนังสือใหม่
            Dim response = client.PostAsync("api/Books", content).Result
            response.EnsureSuccessStatusCode()
        End Using
        ' เรียกฟังก์ชัน LoadBooks อีกครั้งหลังจากเพิ่มหนังสือ
        LoadBooks()
    End Sub

    ' ฟังก์ชันที่จะถูกเรียกเมื่อมีการเริ่มแก้ไขแถวใน GridView
    Protected Sub BooksGridView_RowEditing(sender As Object, e As GridViewEditEventArgs)
        ' ทำให้แถวที่กำลังแก้ไขสามารถแก้ไขได้
        BooksGridView.EditIndex = e.NewEditIndex
        LoadBooks()
    End Sub

    ' ฟังก์ชันที่จะถูกเรียกเมื่อมีการยกเลิกการแก้ไขแถวใน GridView
    Protected Sub BooksGridView_RowCancelingEdit(sender As Object, e As GridViewCancelEditEventArgs)
        ' ยกเลิกการแก้ไขแถว
        BooksGridView.EditIndex = -1
        LoadBooks()
    End Sub

    ' ฟังก์ชันที่จะถูกเรียกเมื่อมีการอัพเดตแถวใน GridView
    Protected Sub BooksGridView_RowUpdating(sender As Object, e As GridViewUpdateEventArgs)
        Dim rowIndex As Integer = e.RowIndex
        ' ดึงค่า Id เพื่อแก้ไขที่ Id นั้นๆ
        Dim id As String = BooksGridView.DataKeys(rowIndex).Value.ToString()
        HiddenFieldRecordID.Value = id
        HiddenFieldOperationType.Value = "Update"

        ' แสดง modal เพื่อยืนยันการอัพเดต
        ScriptManager.RegisterStartupScript(Me, Me.GetType(), "showUpdateModal", "showUpdateModal();", True)
    End Sub

    ' ฟังก์ชันที่จะถูกเรียกเมื่อยืนยันการอัพเดต
    Protected Sub btnConfirmUpdate_Click(sender As Object, e As EventArgs)
        Dim id As String = HiddenFieldRecordID.Value
        Dim rowIndex As Integer = BooksGridView.EditIndex
        Dim row As GridViewRow = BooksGridView.Rows(rowIndex)

        ' สร้างหนังสือที่อัพเดตใหม่จากข้อมูลใน TextBox
        Dim updatedBook As New Book With {
            .Id = id,
            .BookName = DirectCast(row.Cells(1).Controls(0), TextBox).Text,
            .Price = Decimal.Parse(DirectCast(row.Cells(2).Controls(0), TextBox).Text),
            .Category = DirectCast(row.Cells(3).Controls(0), TextBox).Text,
            .Author = DirectCast(row.Cells(4).Controls(0), TextBox).Text
        }

        Using client As New HttpClient()
            ' กำหนดฐานที่อยู่ของ API
            client.BaseAddress = New Uri("http://localhost:5278/")
            client.DefaultRequestHeaders.Accept.Clear()
            client.DefaultRequestHeaders.Accept.Add(New MediaTypeWithQualityHeaderValue("application/json"))

            ' สร้าง JSON จากหนังสือที่อัพเดตใหม่
            Dim json As String = JsonConvert.SerializeObject(updatedBook, New JsonSerializerSettings With {
                .ReferenceLoopHandling = ReferenceLoopHandling.Ignore
            })
            Dim content As New StringContent(json, Encoding.UTF8, "application/json")
            ' ส่งคำร้องขอ PUT ไปยัง API เพื่ออัพเดตหนังสือ
            Dim response = client.PutAsync($"api/Books/{id}", content).Result
            response.EnsureSuccessStatusCode()
        End Using

        ' ปิดการแก้ไขแถว
        BooksGridView.EditIndex = -1
        LoadBooks()
    End Sub

    ' ฟังก์ชันที่จะถูกเรียกเมื่อมีการลบแถวใน GridView
    Protected Sub BooksGridView_RowDeleting(sender As Object, e As GridViewDeleteEventArgs)
        ' ดึงค่าที่แถวที่ถูกเลือก
        Dim rowIndex As Integer = e.RowIndex
        ' เก็บค่า Id
        Dim id As String = BooksGridView.DataKeys(rowIndex).Value.ToString()
        HiddenFieldRecordID.Value = id
        ' แสดงสถานะการดำเนินการเป็น Delete
        HiddenFieldOperationType.Value = "Delete"

        ' แสดง modal เพื่อยืนยันการลบ
        ScriptManager.RegisterStartupScript(Me, Me.GetType(), "showDeleteModal", "showDeleteModal();", True)
    End Sub

    ' ฟังก์ชันที่จะถูกเรียกเมื่อยืนยันการลบ
    Protected Sub btnConfirmDelete_Click(sender As Object, e As EventArgs)
        Dim id As String = HiddenFieldRecordID.Value

        Using client As New HttpClient()
            ' กำหนดฐานที่อยู่ของ API
            client.BaseAddress = New Uri("http://localhost:5278/")
            client.DefaultRequestHeaders.Accept.Clear()
            client.DefaultRequestHeaders.Accept.Add(New MediaTypeWithQualityHeaderValue("application/json"))

            ' ส่งคำร้องขอ DELETE ไปยัง API เพื่อลบหนังสือ
            Dim response = client.DeleteAsync($"api/Books/{id}").Result
            response.EnsureSuccessStatusCode()
        End Using

        ' โหลดข้อมูลหนังสือใหม่
        LoadBooks()
    End Sub

    ' คลาสหนังสือที่ใช้เป็นแบบจำลองข้อมูล
    Public Class Book
        Public Property Id As String
        Public Property BookName As String
        Public Property Price As Decimal
        Public Property Category As String
        Public Property Author As String
    End Class
End Class

```

