# Tài liệu Thiết kế RESTful API - Task & User Management

## 1. Nguyên tắc thiết kế (Design Principles)
*   **Danh từ số nhiều (Plural Nouns):** Sử dụng `/users` và `/tasks` thay vì `/user` hay `/task`.
*   **HTTP Methods chuẩn:** 
    *   `GET`: Truy xuất dữ liệu.
    *   `POST`: Tạo mới dữ liệu.
    *   `PATCH`: Cập nhật một phần dữ liệu (trạng thái, vai trò, người phụ trách).
    *   `DELETE`: Xóa dữ liệu.
*   **Quan hệ thực thể (Relationships):** Sử dụng đường dẫn lồng nhau (Nested Routing) để biểu diễn tài nguyên phụ thuộc, ví dụ: `/users/{userId}/tasks`.

---

## 2. Bảng tổng hợp các Endpoints

| Mục đích | HTTP Method | Endpoint (URL) |
| :--- | :--- | :--- |
| Lấy danh sách người dùng | `GET` | `/users` |
| Lấy danh sách công việc | `GET` | `/tasks` |
| Tạo mới người dùng | `POST` | `/users` |
| Tạo mới công việc | `POST` | `/tasks` |
| Cập nhật vai trò người dùng | `PATCH` | `/users/{id}` |
| Cập nhật trạng thái công việc | `PATCH` | `/tasks/{id}` |
| Xóa người dùng | `DELETE` | `/users/{id}` |
| Xóa công việc | `DELETE` | `/tasks/{id}` |
| Liệt kê công việc của 1 người dùng | `GET` | `/users/{userId}/tasks` |
| Tìm công việc ưu tiên "high" | `GET` | `/tasks?priority=high` |
| Tìm việc ưu tiên "high" của user = 1 | `GET` | `/users/1/tasks?priority=high` (hoặc `/tasks?userId=1&priority=high`) |
| Gắn công việc cho người dùng | `PATCH` | `/tasks/{taskId}/assignee` |

---

## 3. Chi tiết các Endpoints

### 3.1. Quản lý Người dùng (Users)

#### Lấy danh sách người dùng
*   **Method:** `GET`
*   **Endpoint:** `/users`
*   **Response (200 OK):** Mảng chứa các object User.

#### Tạo mới người dùng (Sử dụng `@PostMapping` & `@RequestBody`)
*   **Method:** `POST`
*   **Endpoint:** `/users`
*   **Request Body (JSON):**
    ```json
    {
      "username": "nguyenvana",
      "email": "vana@example.com",
      "role": "MEMBER"
    }
    ```
*   **Response (201 Created):** Trả về thông tin User vừa tạo kèm ID.

#### Cập nhật vai trò của người dùng
*   **Method:** `PATCH`
*   **Endpoint:** `/users/{id}`
*   **Request Body:**
    ```json
    {
      "role": "ADMIN"
    }
    ```

#### Xóa người dùng
*   **Method:** `DELETE`
*   **Endpoint:** `/users/{id}`
*   **Response (204 No Content):** Xóa thành công, không trả về body.

---

### 3.2. Quản lý Công việc (Tasks)

#### Lấy danh sách toàn bộ công việc
*   **Method:** `GET`
*   **Endpoint:** `/tasks`

#### Tạo mới công việc
*   **Method:** `POST`
*   **Endpoint:** `/tasks`
*   **Request Body (JSON):**
    ```json
    {
      "title": "Thiết kế database",
      "description": "Lên cấu trúc các bảng và quan hệ",
      "priority": "high",
      "status": "TODO"
    }
    ```

#### Cập nhật trạng thái công việc
*   **Method:** `PATCH`
*   **Endpoint:** `/tasks/{id}`
*   **Request Body (JSON):**
    ```json
    {
      "status": "IN_PROGRESS"
    }
    ```

#### Xóa công việc
*   **Method:** `DELETE`
*   **Endpoint:** `/tasks/{id}`

---

### 3.3. Tương tác và Lọc Dữ liệu (Relationships & Filtering)

#### Tìm các công việc có mức độ ưu tiên là "high"
*   **Method:** `GET`
*   **Endpoint:** `/tasks?priority=high`
*   **Mô tả:** Sử dụng Query Parameter `priority` để lọc danh sách.

#### Liệt kê toàn bộ công việc của 1 người dùng cụ thể
*   **Method:** `GET`
*   **Endpoint:** `/users/{userId}/tasks`
*   **Mô tả:** Thể hiện rõ mối quan hệ 1-N (Một user có nhiều tasks).

#### Tìm công việc có ưu tiên "high" và giao cho user ID = 1
*   **Method:** `GET`
*   **Endpoint:** `/users/1/tasks?priority=high`
*   *Cách thay thế (Cũng chuẩn REST):* `/tasks?userId=1&priority=high`

#### Gắn công việc cho người dùng (Assign Task)
*   **Method:** `PATCH`
*   **Endpoint:** `/tasks/{taskId}/assignee`
*   **Request Body (JSON):**
    ```json
    {
      "userId": 1
    }
    ```
*   **Mô tả:** Việc gắn task bản chất là cập nhật trường `userId` (Khóa ngoại - Foreign Key) của Task đó.

---

## 4. Kiểm tra dữ liệu (Validation Logic)

Khi cài đặt bằng Spring Boot, cần áp dụng các Annotation Validation trực tiếp vào DTO của `@RequestBody` để đảm bảo tính toàn vẹn dữ liệu trước khi xử lý:

1.  **Bắt buộc nhập:** Sử dụng `@NotBlank` cho các trường `title`, `username`, `email`.
2.  **Định dạng chuẩn:** Sử dụng `@Email` cho trường email của User.
3.  **Giới hạn giá trị:** Sử dụng `@Pattern` hoặc đối tượng `Enum` để đảm bảo `priority` chỉ nhận các giá trị: `low`, `medium`, `high`. Tương tự cho `status`: `TODO`, `IN_PROGRESS`, `DONE`.
4.  **Kích thước độ dài:** Sử dụng `@Size(min = 5, max = 100)` cho tiêu đề công việc.

*Ví dụ áp dụng Validation trong Spring Boot:*
```java
@PostMapping("/tasks")
public ResponseEntity<Task> createTask(@Valid @RequestBody TaskDTO taskDTO) {
    // Logic tạo task...
}
```
