| Environment | Domain                   | Status |
|-------------|--------------------------|--------|
| Dev         | https://dev-airlines.com | Ready  | 
| Prod        | https://airlines.com     | Ready  |


# Content
1. [API search flights](#api-search-flights)
2. [API search booking](#api-booking-flights)
3. [API get history of booking](#api-get-history-of-booking)

# API search flights

> Service Specification

| Title          | Value                           |
|----------------|---------------------------------|
| Endpoint       | https://{domain}/api/v1/flights |
| Description    | API tìm kiếm vé máy bay         |
| Method         | GET                             |
| Authentication | Token                           |

> Request

Request Body: N/A

Request Params:

| Field          | Type    | Required | Description                                                            |
|:---------------|:--------|:---------|:-----------------------------------------------------------------------|
| origin         | String  | Yes      | Tên thành phố khởi hành                                                |
| destination    | String  | Yes      | Tên thành phố đến                                                      |
| departure_date | String  | No       | Ngày khởi hành (YYYY-MM-DD)                                            |
| return_date    | String  | No       | Ngày quay về (YYYY-MM-DD)                                              |
| passengers     | Integer | No       | Số lượng người lớn và trẻ em                                           |
| sort           | String  | No       | Sắp xếp kết quả theo giá (price), thời gian khởi hành (departure_time) |
| size           | Integer | No       | Số lượng kết quả tối đa trả về (default = 10)                          |
| page           | Integer | No       | Số trang cần xem  (default = 1)                                        |

Request Header: N/A

> Response

| Field               | Type   | Description                   |
|:--------------------|:-------|:------------------------------|
| airline             | String | Tên hãng hàng không           |
| flight_number       | String | Số hiệu chuyến bay            |
| origin_airport      | Json   | Mã và Tên sân bay khởi hành   |
| destination_airport | Json   | Mã và Tên sân bay đến         |
| departure_time      | String | Thời gian khởi hành           |
| arrival_time        | String | Thời gian đến                 |
| duration            | String | Thời gian bay                 |
| price               | Json   | Giá vé                        |
| links               | String | Liên kết đến trang web đặt vé | 

ErrorResponse

| No | Key        | Type   | Description          |
|----|------------|--------|----------------------|
| 3  | error_code | String | Mã lỗi               |
| 4  | message    | String | Mô tả                |
| 6  | trace_id   | String | Id Trace để kiểm tra |

> Sample request

```
curl --location --request GET '{domain}/api/v1/flights?origin=SGN&destination=HAN&departure_date=2023-12-24' \
--header 'Authorization: Bearer {token}' 
```

> Sample response

Success: Http status code = 200

```
[
  {
    "airline": "Vietnam Airlines",
    "flight_number": "VN123",
    "origin_airport": {
      "code": "SGN",
      "name": "Sân bay quốc tế Nội Bài"
    },
    "destination_airport": {
      "code": "HAN",
      "name": "Sân bay quốc tế Tân Sơn Nhất"
    },
    "departure_time": "2023-12-24T10:00:00",
    "arrival_time": "2023-12-24T11:30:00",
    "duration": "1 giờ 30 phút",
    "price": {
      "amount": 1200000,
      "currency": "VND"
    },
    "links": "https://www.vietnamairlines.com/vi-vn/dat-ve"
  }
]
```
Error: Http status code = 400. Các lỗi validate
```
{
  "error_code": "validation_error",
  "message": "origin not found"
}
```
```
{
  "error_code": "validation_error",
  "message": "destination not found"
}
```
```
{
  "error_code": "validation_error",
  "message": "parameter departure_date is an invalid format"
}
```
```
{
  "error_code": "validation_error",
  "message": "parameter return_date is an invalid format"
}
```
Error: Http status code = 401. Token lỗi
```
{
  "error_code": "unauthorized",
  "message": "Invalid token"
}
```

# API booking flights

> Service Specification

| Title          | Value                                    |
|----------------|------------------------------------------|
| Endpoint       | https://{domain}/api/v1/flights/booking  |
| Description    | API đặt vé  máy bay                      |
| Method         | POST                                     |
| Authentication | Token                                    |

> Request

Request Body: 

| Field           | Type    | Required | Description          |
|:----------------|:--------|:---------|:---------------------|
| flight_id       | String  | Yes      | ID của chuyến bay    |
| passenger_adult | integer | Yes      | Số lượng người lớn   |
| passenger_child | integer | No       | Số lượng trẻ em      |
| contact_name    | String  | Yes      | Tên người đặt vé     |
| contact_email   | String  | Yes      | Email đặt vé         |
| contact_phone   | String  | Yes      | Số điện thoại đặt vé |

Request Params: N/A

Request Header: N/A

> Response

| Field        | Type     | Description                             |
|:-------------|:---------|:----------------------------------------|
| status       | String   | Trạng thái đặt vé,                      |
| message      | String   | Thông báo chi tiết về trạng thái đặt vé |

ErrorResponse

| No | Key        | Type   | Description          |
|----|------------|--------|----------------------|
| 3  | error_code | String | Mã lỗi               |
| 4  | message    | String | Mô tả                |
| 6  | trace_id   | String | Id Trace để kiểm tra |

> Sample request

```
curl --location --request GET 'https://{domain}/api/v1/flights/booking' \
--header 'Authorization: Bearer {token}' \
--data '{
  "flight_id": 12345,
  "passenger_adult": "1",
  "passenger_child": "2",
  "contact_name": "Nguyễn Văn A",
  "contact_email": "nguyenvana@example.com",
  "contact_phone": "0912345678"
  
}'
```

> Sample response

Success: Http status code = 200

```
{
  "status": "success",
  "message": "Đặt vé thành công"
}
```
Error: Http status code = 400. Các lỗi validate
```
{
  "error_code": "validation_error",
  "message": "parameter flight_id is an invalid format"
}
```
Error: Http status code = 401. Token lỗi
```
{
  "error_code": "unauthorized",
  "message": "Invalid token"
}
```
Error: Http status code = 404. Not Found
```
{
  "error_code": "not_found",
  "message": "Chuyến bay không tồn tại"
}
```
Error: Http status code = 500. Lỗi máy chủ
```
{
  "error_code": "internal_server_error",
  "message": "Lỗi máy chủ khi xử lý yêu cầu"
}
```


# API Get History of Booking

> Service Specification

| Title          | Value                                     |
|----------------|-------------------------------------------|
| Endpoint       | https://{domain}/api/v1/flights/history   |
| Description    | Lấy lịch sử đặt vé máy bay của người dùng |
| Method         | GET                                       |
| Authentication | Token                                     |

> Request

Request Body: N/A

Request Params:

| Field          | Type    | Required | Description                                                            |
|:---------------|:--------|:---------|:-----------------------------------------------------------------------|
| size           | Integer | No       | Số lượng kết quả tối đa trả về (default = 10)                          |
| page           | Integer | No       | Số trang cần xem  (default = 1)                                        |


Request Header: N/A

> Response

| Field          | Type    | Description                                        |
|:---------------|:--------|:---------------------------------------------------|
| bookings       | Json    | Danh sách các chuyến bay đã đặt                    |
| flight_id      | String  | ID của chuyến bay                                  |
| origin         | String  | Mã sân bay khởi hành                               |
| destination    | String  | Mã sân bay đến                                     |
| departure_date | String  | Ngày khởi hành (Định dạng YYYY-MM-DD)              |
| arrival_date   | String  | Ngày đến (Định dạng YYYY-MM-DD)                    |
| price          | Integer | Giá vé                                             |
| status         | String  | Trạng thái đặt vé                                  |
| created_at     | String  | Thời gian đặt vé  (Định dạng YYYY-MM-DDTHH:mm:ssZ) |
| meta           | Json    | Thông tin tổng quan về lịch sử đặt vé              |
| total_bookings | Integer | total_bookings                                     |
| total_price    | Integer | total_price                                        |
ErrorResponse

| No | Key        | Type   | Description          |
|----|------------|--------|----------------------|
| 3  | error_code | String | Mã lỗi               |
| 4  | message    | String | Mô tả                |
| 6  | trace_id   | String | Id Trace để kiểm tra |

> Sample request

```
curl --location --request GET 'https://{domain}/api/v1/flights/history' \
--header 'Authorization: Bearer {token}' \
```

> Sample response

Success: Http status code = 200

```
{
  "bookings": [
    {
      "flight_id": 12345,
      "origin": "SGN",
      "destination": "HAN",
      "departure_date": "2023-12-24",
      "arrival_date": "2023-12-25",
      "price": 1200000,
      "status": "success",
      "created_at": "2023-12-24T10:00:00Z"
    },
    {
      "flight_id": 67890,
      "origin": "HAN",
      "destination": "SGN",
      "departure_date": "2023-12-26",
      "arrival_date": "2023-12-27",
      "price": 2400000,
      "status": "failed",
      "created_at": "2023-12-25T10:00:00Z"
    }
  ],
  "meta": {
    "total_bookings": 2,
    "total_price": 3600000
  }
}
```


Error: Http status code = 401. Token lỗi
```
{
  "error_code": "unauthorized",
  "message": "Invalid token"
}
```

Error: Http status code = 500. Lỗi máy chủ
```
{
  "error_code": "internal_server_error",
  "message": "Lỗi máy chủ khi xử lý yêu cầu"
}
```