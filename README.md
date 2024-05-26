# Hướng dẫn code go theo phong cách Uber

- [Giới thiệu](#introduction)
- [Hướng dẫn](#guidelines)
  - [Pointers to Interfaces](#pointers-to-interfaces)
  - [Xác minh sự tuân thủ Interfaces](#verify-interface-compliance)
  - [Receivers and Interfaces](#receivers-and-interfaces)
  - [Muexes có giá trị bằng 0 là hợp lệ](#zero-value-mutexes-are-valid)
  - [Sao chép Slices and Maps tại Boundaries](#copy-slices-and-maps-at-boundaries)
  - [Dùng Defer để dọn dẹp](#defer-to-clean-up)
  - [Kích thước Channel là 1 hoặc None](#channel-size-is-one-or-none)
  - [Bắt đầu Enums tại 1](#start-enums-at-one)
  - [Sử `"time"` để xử lý thời gian](#use-time-to-handle-time)
  - [Lỗi](#errors)
    - [Các loại lỗi](#error-types)
    - [Gói lỗi](#error-wrapping)
    - [Lỗi đặt tên](#error-naming)
    - [Xử lý Lỗi Một Lần](#handle-errors-once)
  - [Xử Lý Lỗi Kiểu Assertion](#handle-type-assertion-failures)
  - [Không Sử Dụng Panic](#dont-panic)
  - [Sử Dụng go.uber.org/atomic](#use-gouberorgatomic)
  - [Tránh Sử Dụng Biến Toàn Cục Thay Đổi](#avoid-mutable-globals)
  - [Tránh Nhúng Các Loại vào Cấu Trúc Công Khai](#avoid-embedding-types-in-public-structs)
  - [Tránh Sử Dụng Tên Được Xây Dựng Sẵn](#avoid-using-built-in-names)
  - [Tránh `init()`](#avoid-init)
  - [Thoát Ở Main](#exit-in-main)
    - [Thoát Một Lần](#exit-once)
  - [Sử Dụng Thẻ Trường Trong Các Cấu Trúc Được Unmarshaled](#use-field-tags-in-marshaled-structs)
  - [Không Gửi và Quên Goroutines](#dont-fire-and-forget-goroutines)
    - [Chờ Goroutines Thoát](#wait-for-goroutines-to-exit)
    - [Không Có Goroutines Trong `init()`](#no-goroutines-in-init)
- [Hiệu Suất](#performance)
  - [Ưu Tiên strconv hơn fmt](#prefer-strconv-over-fmt)
  - [Tránh Chuyển Đổi Chuỗi sang Byte Lặp Lại](#avoid-repeated-string-to-byte-conversions)
  - [Ưu Tiên Xác Định Khả Năng của Container](#prefer-specifying-container-capacity)
- [Kiểu](#style)
  - [Tránh Các Dòng Quá Dài](#avoid-overly-long-lines)
  - [Duy Trì Sự Nhất Quán](#be-consistent)
  - [Nhóm Các Khai Báo Tương Tự](#group-similar-declarations)
  - [Thứ Tự Nhóm Nhập](#import-group-ordering)
  - [Tên Gói](#package-names)
  - [Tên Hàm](#function-names)
  - [Bí Danh Nhập](#import-aliasing)
  - [Nhóm và Sắp Xếp Hàm](#function-grouping-and-ordering)
  - [Giảm Lồng Ghép](#reduce-nesting)
  - [Else Không Cần Thiết](#unnecessary-else)
  - [Khai Báo Biến Cấp Cao](#top-level-variable-declarations)
  - [Thêm Tiền Tố Cho Các Biến Toàn Cục Không Được Xuất \_](#prefix-unexported-globals-with-_)
  - [Nhúng Trong Cấu Trúc](#embedding-in-structs)
  - [Khai Báo Biến Cục Bộ](#local-variable-declarations)
  - [nil Là Một Slice Hợp Lệ](#nil-is-a-valid-slice)
  - [Giảm Phạm Vi của Biến](#reduce-scope-of-variables)
  - [Tránh Tham Số Trần](#avoid-naked-parameters)
  - [Sử Dụng Chuỗi Văn Bản Thô Để Tránh Escape](#use-raw-string-literals-to-avoid-escaping)
  - [Khởi Tạo Các Cấu Trúc](#initializing-structs)
    - [Sử Dụng Tên Trường để Khởi Tạo Cấu Trúc](#use-field-names-to-initialize-structs)
    - [Bỏ Qua Các Trường Giá Trị Zero trong Cấu Trúc](#omit-zero-value-fields-in-structs)
    - [Sử Dụng `var` Cho Cấu Trúc Giá Trị Zero](#use-var-for-zero-value-structs)
    - [Khởi Tạo Các Tham Chiếu Cấu Trúc](#initializing-struct-references)
  - [Khởi tạo Maps](#initializing-maps)
  - [Định Dạng Chuỗi Ngoài Printf](#format-strings-outside-printf)
  - [Đặt Tên Các Hàm Theo Phong Cách Printf](#naming-printf-style-functions)
- [Mẫu Thiết Kế](#patterns)
  - [Bảng Kiểm Tra](#test-tables)
  - [Tuỳ Chọn Chức Năng](#functional-options)
- [Kiểm Tra Cú Pháp](#linting)

## Introduction

Kiểu là các quy ước chi phối mã của chúng tôi. Thuật ngữ kiểu hơi bị dùng sai, vì những quy ước này không chỉ bao gồm việc định dạng tệp nguồn—gofmt xử lý việc đó cho chúng ta.

Mục tiêu của hướng dẫn này là quản lý sự phức tạp bằng cách mô tả chi tiết những điều nên làm và không nên làm khi viết mã Go tại Uber. Những quy tắc này tồn tại để giữ cho cơ sở mã nguồn dễ quản lý trong khi vẫn cho phép các kỹ sư sử dụng các tính năng của ngôn ngữ Go một cách hiệu quả.

Hướng dẫn này ban đầu được tạo bởi [Prashant Varanasi](https://github.com/prashantv) và [Simon Newton](https://github.com/nomis52) như một cách để giúp một số đồng nghiệp nắm bắt nhanh với việc sử dụng Go. Qua các năm, nó đã được chỉnh sửa dựa trên phản hồi từ những người khác.

Tài liệu này ghi lại các quy ước idiomatic trong mã Go mà chúng tôi tuân theo tại Uber. Nhiều trong số này là hướng dẫn chung cho Go, trong khi những quy tắc khác mở rộng dựa trên các nguồn tài liệu bên ngoài:

1. [Effective Go](https://go.dev/doc/effective_go)
2. [Go Common Mistakes](https://go.dev/wiki/CommonMistakes)
3. [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments)

Chúng tôi nhắm đến việc các mẫu mã đều chính xác cho hai phiên bản nhỏ gần đây nhất của các bản phát hành Go. [releases](https://go.dev/doc/devel/release).

Tất cả mã phải không có lỗi khi chạy qua `golint` và `go vet`. Chúng tôi khuyến nghị thiết lập trình soạn thảo của bạn để:

- Chạy `goimports` khi lưu
- Chạy `golint` và `go vet` để kiểm tra lỗi

Bạn có thể tìm thấy thông tin về hỗ trợ trình soạn thảo cho các công cụ Go tại đây:
https://go.dev/wiki/IDEsAndTextEditorPlugins

## Guidelines

### Pointers to Interfaces

Bạn gần như không bao giờ cần một pointer tới một interface. Bạn nên truyền interfaces như là values—dữ liệu cơ bản vẫn có thể là một pointer.

Một interface bao gồm hai trường:

1. Một pointer đến một thông tin cụ thể về kiểu. Bạn có thể nghĩ về điều này như là "type."
2. Data pointer. Nếu dữ liệu được lưu trữ là một pointer, nó được lưu trữ trực tiếp. Nếu dữ liệu được lưu trữ là một value, thì một pointer tới value đó được lưu trữ.

Nếu bạn muốn các phương thức của interface thay đổi dữ liệu cơ bản, bạn phải sử dụng một pointer.

### Verify Interface Compliance

Xác minh tuân thủ interface tại thời điểm biên dịch khi thích hợp. Điều này bao gồm:

- Các loại được xuất (exported types) yêu cầu thực hiện các interfaces cụ thể như một phần của hợp đồng API của chúng
- Các loại được xuất hoặc không được xuất (exported or unexported types) là một phần của một tập hợp các loại thực hiện cùng một interface
- Các trường hợp khác mà việc vi phạm một interface sẽ gây hại cho người dùng

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
type Handler struct {
  // ...
}



func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  ...
}
```

</td><td>

```go
type Handler struct {
  // ...
}

var _ http.Handler = (*Handler)(nil)

func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

</td></tr>
</tbody></table>

Câu lệnh `var _ http.Handler = (*Handler)(nil)` sẽ không thể biên dịch nếu `*Handler` không còn phù hợp với interface `http.Handler`.

Phía bên phải của phép gán nên là giá trị zero của kiểu được khẳng định. Điều này là `nil` đối với các kiểu pointer (như `*Handler`), slices và maps, và một struct rỗng đối với các kiểu struct.

```go
type LogHandler struct {
  h   http.Handler
  log *zap.Logger
}

var _ http.Handler = LogHandler{}

func (h LogHandler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

### Receivers and Interfaces

Các phương thức với value receivers có thể được gọi trên cả pointers lẫn values. Các phương thức với pointer receivers chỉ có thể được gọi trên pointers hoặc [addressable values](https://go.dev/ref/spec#Method_values).

Ví dụ,

```go
type S struct {
  data string
}

func (s S) Read() string {
  return s.data
}

func (s *S) Write(str string) {
  s.data = str
}

// Chúng ta không thể lấy pointers đến các giá trị được lưu trữ trong maps, vì chúng không phải là các addressable values.
sVals := map[int]S{1: {"A"}}

// Chúng ta có thể gọi Read trên các giá trị được lưu trữ trong map vì Read
// có một value receiver, không yêu cầu giá trị phải là addressable.
sVals[1].Read()

// Chúng ta không thể gọi Write trên các giá trị được lưu trữ trong map vì Write
// có một pointer receiver, và không thể lấy một pointer
// đến một giá trị được lưu trữ trong map.
//
//  sVals[1].Write("test")

sPtrs := map[int]*S{1: {"A"}}

// Bạn có thể gọi cả Read và Write nếu map lưu trữ pointers,
// vì pointers bản chất là addressable.
sPtrs[1].Read()
sPtrs[1].Write("test")
```

Tương tự, một interface có thể được đáp ứng bởi một pointer, ngay cả khi phương thức đó có một value receiver.

```go
type F interface {
  f()
}

type S1 struct{}

func (s S1) f() {}

type S2 struct{}

func (s *S2) f() {}

s1Val := S1{}
s1Ptr := &S1{}
s2Val := S2{}
s2Ptr := &S2{}

var i F
i = s1Val
i = s1Ptr
i = s2Ptr

// Đoạn mã sau không thể biên dịch, vì s2Val là một value, và không có value receiver cho f.
//   i = s2Val
```

"Effective Go" có một bài viết tốt về [Pointers vs. Values](https://go.dev/doc/effective_go#pointers_vs_values).

### Zero-value Mutexes are Valid

Giá trị zero của `sync.Mutex` và `sync.RWMutex` là hợp lệ, vì vậy bạn gần như không bao giờ cần một con trỏ đến một mutex.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
mu := new(sync.Mutex)
mu.Lock()
```

</td><td>

```go
var mu sync.Mutex
mu.Lock()
```

</td></tr>
</tbody></table>

If you use a struct by pointer, then the mutex should be a non-pointer field on
it. Do not embed the mutex on the struct, even if the struct is not exported.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type SMap struct {
  sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

</td><td>

```go
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

</td></tr>

<tr><td>

Trường `Mutex`, và các phương thức `Lock` và `Unlock` đều không cố ý là một phần của API được xuất của SMap.

</td><td>

`Mutex` và các phương thức của nó là các chi tiết cài đặt của `SMap` được ẩn đi khỏi những người gọi của nó.

</td></tr>
</tbody></table>

### Copy Slices and Maps at Boundaries

Slices và maps chứa các con trỏ đến dữ liệu cơ bản, vì vậy hãy cẩn thận trong các tình huống khi cần phải sao chép chúng.

#### Receiving Slices and Maps

Hãy nhớ rằng người dùng có thể sửa đổi một map hoặc slice mà bạn nhận làm đối số nếu bạn lưu trữ một tham chiếu đến nó.

<table>
<thead><tr><th>Không nên</th> <th>Nên</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Bạn có ý muốn sửa đổi d1.trips chứ?
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// Bây giờ chúng ta có thể sửa đổi trips[0] mà không ảnh hưởng đến d1.trips.
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

#### Returning Slices and Maps

Tương tự, hãy cẩn thận với việc người dùng sửa đổi maps hoặc slices, tiết lộ trạng thái nội bộ.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Hàm Snapshot trả về các thống kê hiện tại.
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot không còn được bảo vệ bởi mutex nữa, vì vậy bất kỳ
// truy cập nào vào snapshot đều có thể gặp phải các cuộc đua dữ liệu.
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Bản sao của Snapshot bây giờ.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

### Defer to Clean Up

Sử dụng defer để dọn dẹp tài nguyên như tệp và khóa.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// dễ bỏ sót việc mở khóa do có nhiều câu lệnh return
```

</td><td>

```go
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// more readable
```

</td></tr>
</tbody></table>

Defer có một chi phí rất nhỏ và chỉ nên được tránh nếu bạn có thể chứng minh rằng thời gian thực thi của hàm của bạn có thứ tự trong các nanogiây. Sự dễ đọc khi sử dụng defer đáng giá với chi phí nhỏ nhất của việc sử dụng chúng. Điều này đặc biệt đúng đối với các phương thức lớn có nhiều hơn là các truy cập bộ nhớ đơn giản, nơi các tính toán khác quan trọng hơn là `defer`.

### Channel Size is One or None

Các channels thường nên có kích thước là một hoặc không được đệm. Theo mặc định, các channels không được đệm và có kích thước là không. Bất kỳ kích thước khác phải chịu sự kiểm tra mức độ cao. Xem xét cách kích thước được xác định, những gì ngăn kênh khỏi bị đầy dưới tải và chặn việc ghi, và điều gì xảy ra khi điều này xảy ra.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
// Có lẽ đủ cho mọi người!
c := make(chan int, 64)
```

</td><td>

```go
// Kích thước là một
c := make(chan int, 1) // hoặc
// Unbuffered channel, kích thước là không
c := make(chan int)
```

</td></tr>
</tbody></table>

### Start Enums at One

Cách tiêu chuẩn để giới thiệu các enumerations trong Go là khai báo một kiểu tùy chỉnh và một nhóm `const` với `iota`. Vì biến có giá trị mặc định là 0, bạn thường nên bắt đầu enum của mình từ một giá trị không phải là 0.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

Có những trường hợp mà việc sử dụng giá trị zero là hợp lý, ví dụ khi trường hợp giá trị zero là hành vi mặc định mong muốn.

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

<!-- TODO: phần về các phương thức String cho các enum -->

### Use `"time"` to handle time

Thời gian là một vấn đề phức tạp. Những giả định sai lầm thường gặp về thời gian bao gồm những điều sau đây.

1. Một ngày có 24 giờ
2. Một giờ có 60 phút
3. Một tuần có 7 ngày
4. Một năm có 365 ngày
5. [Và rất nhiều điều khác](https://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time)

Ví dụ, _1_ có nghĩa là thêm 24 giờ vào một thời điểm cụ thể không luôn luôn tạo ra một ngày mới trên lịch.

Do đó, luôn luôn sử dụng [`"time"`](https://pkg.go.dev/time) package khi làm việc với thời gian vì nó giúp xử lý những giả định sai lầm này một cách an toàn và chính xác hơn.

#### Use `time.Time` for instants of time

Hãy sử dụng [`time.Time`](https://pkg.go.dev/time#Time) khi làm việc với các thời điểm cụ thể, và các phương thức trên `time.Time` khi so sánh, thêm hoặc trừ thời gian.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
func isActive(now, start, stop int) bool {
  return start <= now && now < stop
}
```

</td><td>

```go
func isActive(now, start, stop time.Time) bool {
  return (start.Before(now) || start.Equal(now)) && now.Before(stop)
}
```

</td></tr>
</tbody></table>

#### Use `time.Duration` for periods of time

Sử dụng [`time.Duration`](https://pkg.go.dev/time#Duration) khi làm việc với các khoảng thời gian.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
func poll(delay int) {
  for {
    // ...
    time.Sleep(time.Duration(delay) * time.Millisecond)
  }
}

poll(10) // liệu đó có phải là giây hay mili-giây không?
```

</td><td>

```go
func poll(delay time.Duration) {
  for {
    // ...
    time.Sleep(delay)
  }
}

poll(10*time.Second)
```

</td></tr>
</tbody></table>

Quay trở lại ví dụ về việc thêm 24 giờ vào một thời điểm cụ thể, phương thức chúng ta sử dụng để thêm thời gian phụ thuộc vào ý định. Nếu chúng ta muốn cùng một thời gian trong một ngày tiếp theo trên lịch, chúng ta nên sử dụng [`Time.AddDate`](https://pkg.go.dev/time#Time.AddDate). Tuy nhiên, nếu chúng ta muốn một thời điểm chắc chắn là 24 giờ sau thời điểm trước đó, chúng ta nên sử dụng [`Time.Add`](https://pkg.go.dev/time#Time.Add).

```go
newDay := t.AddDate(0 /* years */, 0 /* months */, 1 /* days */)
maybeNewDay := t.Add(24 * time.Hour)
```

#### Use `time.Time` and `time.Duration` with external systems

Trong tương tác với các hệ thống bên ngoài, hãy sử dụng `time.Duration` và `time.Time` khi có thể. Ví dụ:

- Các cờ dòng lệnh: [`flag`](https://pkg.go.dev/flag) hỗ trợ `time.Duration` thông qua [`time.ParseDuration`](https://pkg.go.dev/time#ParseDuration).
- JSON: [`encoding/json`](https://pkg.go.dev/encoding/json) hỗ trợ mã hóa `time.Time` thành chuỗi [RFC 3339](https://tools.ietf.org/html/rfc3339) thông qua phương thức [`UnmarshalJSON` method](https://pkg.go.dev/time#Time.UnmarshalJSON).
- SQL: [`database/sql`](https://pkg.go.dev/database/sql) hỗ trợ chuyển đổi cột `DATETIME` hoặc `TIMESTAMP` thành `time.Time` và ngược lại nếu driver cơ sở dữ liệu hỗ trợ.
- YAML: [`gopkg.in/yaml.v2`](https://pkg.go.dev/gopkg.in/yaml.v2) hỗ trợ `time.Time` dưới dạng chuỗi [RFC 3339](https://tools.ietf.org/html/rfc3339) và `time.Duration` thông qua [`time.ParseDuration`](https://pkg.go.dev/time#ParseDuration).

Khi không thể sử dụng `time.Duration` trong các tương tác này, hãy sử dụng `int` hoặc `float64` và bao gồm đơn vị trong tên của trường.

Ví dụ, vì `encoding/json` không hỗ trợ `time.Duration`, đơn vị được bao gồm trong tên của trường.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
// {"interval": 2}
type Config struct {
  Interval int `json:"interval"`
}
```

</td><td>

```go
// {"intervalMillis": 2000}
type Config struct {
  IntervalMillis int `json:"intervalMillis"`
}
```

</td></tr>
</tbody></table>

Khi không thể sử dụng `time.Time` trong các tương tác này, trừ khi đã thống nhất về một phương án thay thế, hãy sử dụng `string` và định dạng thời gian dưới dạng được xác định trong [RFC 3339](https://tools.ietf.org/html/rfc3339). Định dạng này được mặc định sử dụng bởi [`Time.UnmarshalText`](https://pkg.go.dev/time#Time.UnmarshalText) và có sẵn để sử dụng trong `Time.Format` và `time.Parse` thông qua [`time.RFC3339`](https://pkg.go.dev/time#RFC3339).

Mặc dù điều này thường không phải là một vấn đề trong thực tế, nhưng hãy nhớ rằng gói `"time"` không hỗ trợ phân tích cú pháp các dấu thời gian có giây nhảy ([8728](https://github.com/golang/go/issues/8728)), cũng như không tính toán cho các giây nhảy trong các phép tính ([15190](https://github.com/golang/go/issues/15190)). Nếu bạn so sánh hai thời điểm cụ thể, sự khác biệt sẽ không bao gồm các giây nhảy có thể đã xảy ra giữa hai thời điểm đó.

### Errors

#### Error Types

Có một số lựa chọn để khai báo lỗi.
Hãy xem xét các điều sau trước khi chọn lựa chọn phù hợp nhất cho trường hợp sử dụng của bạn.

- Người gọi cần phải khớp với lỗi để họ có thể xử lý nó không?
  Nếu có, chúng ta phải hỗ trợ các hàm [`errors.Is`](https://pkg.go.dev/errors#Is) hoặc [`errors.As`](https://pkg.go.dev/errors#As) bằng cách khai báo một biến lỗi ở cấp độ cao nhất hoặc một kiểu tùy chỉnh.
- Thông báo lỗi có phải là một chuỗi tĩnh không,
  hay nó là một chuỗi động cần thông tin ngữ cảnh không?
  Đối với trường hợp đầu tiên, chúng ta có thể sử dụng [`errors.New`](https://pkg.go.dev/errors#New), nhưng đối với trường hợp thứ hai chúng ta phải
  sử dụng [`fmt.Errorf`](https://pkg.go.dev/fmt#Errorf) hoặc một kiểu lỗi tùy chỉnh.
- Chúng ta có đang truyền lỗi mới được trả về bởi một hàm con không?
  Nếu có, xem [section on error wrapping](#error-wrapping).

| Error matching? | Error Message | Guidance                                                           |
| --------------- | ------------- | ------------------------------------------------------------------ |
| No              | static        | [`errors.New`](https://pkg.go.dev/errors#New)                      |
| No              | dynamic       | [`fmt.Errorf`](https://pkg.go.dev/fmt#Errorf)                      |
| Yes             | static        | top-level `var` with [`errors.New`](https://pkg.go.dev/errors#New) |
| Yes             | dynamic       | custom `error` type                                                |

Ví dụ,
Sử dụng [`errors.New`](https://pkg.go.dev/errors#New) cho một lỗi có một chuỗi tĩnh.
Xuất lỗi này như một biến để hỗ trợ khớp nó với `errors.Is`
nếu người gọi cần khớp và xử lý lỗi này.

<table>
<thead><tr><th>No error matching</th><th>Error matching</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

if err := foo.Open(); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if errors.Is(err, foo.ErrCouldNotOpen) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Đối với một lỗi có một chuỗi động,
sử dụng [`fmt.Errorf`](https://pkg.go.dev/fmt#Errorf) nếu người gọi không cần khớp với nó,
và một `error` tùy chỉnh nếu người gọi cần phải khớp với nó.

<table>
<thead><tr><th>No error matching</th><th>Error matching</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

// package bar

if err := foo.Open("testfile.txt"); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

func Open(file string) error {
  return &NotFoundError{File: file}
}


// package bar

if err := foo.Open("testfile.txt"); err != nil {
  var notFound *NotFoundError
  if errors.As(err, &notFound) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Lưu ý rằng nếu bạn xuất các biến lỗi hoặc kiểu từ một gói,
chúng sẽ trở thành một phần của API công khai của gói.

#### Error Wrapping

Có ba lựa chọn chính để truyền tiếp các lỗi nếu một cuộc gọi thất bại:

- Trả về lỗi gốc nguyên vẹn
- Thêm ngữ cảnh với `fmt.Errorf` và động từ `%w`
- Thêm ngữ cảnh với `fmt.Errorf` và động từ `%v`

Trả về lỗi gốc nguyên vẹn nếu không có ngữ cảnh bổ sung nào cần thêm.
Điều này giữ nguyên kiểu và thông báo lỗi ban đầu.
Điều này phù hợp cho các trường hợp khi thông báo lỗi gốc
đã cung cấp đủ thông tin để theo dõi nó đến từ đâu.

Nếu không, thêm ngữ cảnh vào thông báo lỗi nơi có thể
để thay vì một lỗi mơ hồ như "kết nối từ chối",
bạn nhận được các thông báo lỗi hữu ích hơn như "gọi dịch vụ foo: kết nối từ chối".

Sử dụng `fmt.Errorf` để thêm ngữ cảnh vào các lỗi của bạn,
chọn giữa các từ khóa `%w` hoặc `%v`
dựa trên việc người gọi có nên có thể
khớp và trích xuất nguyên nhân cơ bản.

- Sử dụng `%w` nếu người gọi cần có quyền truy cập vào lỗi cơ bản.
  Đây là một lựa chọn mặc định tốt cho hầu hết các lỗi đã được bọc,
  nhưng hãy nhớ rằng người gọi có thể bắt đầu phụ thuộc vào hành vi này.
  Vì vậy đối với các trường hợp mà lỗi được bọc là một `var` hoặc kiểu đã biết,
  hãy tài liệu hóa và kiểm thử nó như một phần của hợp đồng hàm của bạn.
- Sử dụng `%v` để ẩn lỗi cơ bản.
  Người gọi sẽ không thể khớp được nó,
  nhưng bạn có thể chuyển sang `%w` trong tương lai nếu cần.

Khi thêm ngữ cảnh vào các lỗi trả về, hãy giữ ngữ cảnh ngắn gọn bằng cách tránh
các cụm từ như "thất bại" (failed to), điều này nêu ra điều đã rõ và chồng chất khi lỗi
truyền lên qua các tầng của ngăn xếp:

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "failed to create new store: %w", err)
}
```

</td><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "new store: %w", err)
}
```

</td></tr><tr><td>

```plain
failed to x: failed to y: failed to create new store: the error
```

</td><td>

```plain
x: y: new store: the error
```

</td></tr>
</tbody></table>

Tuy nhiên, khi lỗi được gửi đến một hệ thống khác, nó nên rõ ràng rằng thông điệp là một lỗi (ví dụ như một thẻ err hoặc tiền tố "Failed" trong các nhật ký).

Xem thêm [Đừng chỉ kiểm tra lỗi, xử lý chúng một cách dễ dàng.](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully).

#### Error Naming

Đối với các giá trị lỗi được lưu trữ như biến toàn cục,
sử dụng tiền tố Err hoặc err tùy thuộc vào việc chúng có được xuất khẩu hay không.
Hướng dẫn này mạnh mẽ hơn [Tiền tố biến toàn cục chưa được xuất khẩu với \_](#prefix-unexported-globals-with-_).

```go
var (
  // Hai lỗi sau đây được xuất khẩu
  // để người dùng của gói này có thể khớp chúng
  // với errors.Is.

  ErrBrokenLink = errors.New("link is broken")
  ErrCouldNotOpen = errors.New("could not open")

  // Lỗi này không được xuất khẩu vì
  // chúng tôi không muốn làm cho nó trở thành một phần của API công khai của chúng tôi.
  // Tuy nhiên, chúng tôi vẫn có thể sử dụng nó bên trong gói
  // với errors.Is.

  errNotFound = errors.New("not found")
)
```

For custom error types, use the suffix `Error` instead.

```go
// Tương tự, lỗi này được xuất khẩu
// để người dùng của gói này có thể khớp với nó
// với errors.As.

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

// Và lỗi này không được xuất khẩu vì
// chúng tôi không muốn làm cho nó trở thành một phần của API công khai.
// Tuy nhiên, chúng tôi vẫn có thể sử dụng nó bên trong gói
// với errors.As.

type resolveError struct {
  Path string
}

func (e *resolveError) Error() string {
  return fmt.Sprintf("resolve %q", e.Path)
}
```

#### Handle Errors Once

Khi người gọi nhận được lỗi từ hàm được gọi, nó có thể xử lý lỗi theo nhiều cách khác nhau tùy thuộc vào những gì nó biết về lỗi.

Các cách này bao gồm, nhưng không giới hạn:

- nếu hợp đồng của hàm được gọi định nghĩa các lỗi cụ thể,
  khớp lỗi với `errors.Is` hoặc `errors.As`
  và xử lý các nhánh khác nhau
- nếu lỗi có thể khôi phục được,
  ghi log lỗi và xử lý một cách nhẹ nhàng
- nếu lỗi đại diện cho điều kiện thất bại cụ thể của miền,
  trả về một lỗi được định nghĩa rõ ràng
- trả về lỗi, có thể [wrapped](#error-wrapping) hoặc giữ nguyên

Bất kể cách người gọi xử lý lỗi như thế nào,
nó nên xử lý mỗi lỗi chỉ một lần.
Người gọi không nên, ví dụ, ghi log lỗi và sau đó trả về lỗi đó,
vì _người gọi_ của nó cũng có thể xử lý lỗi.

Ví dụ, hãy xem xét các trường hợp sau:

<table>
<thead><tr><th>Description</th><th>Code</th></tr></thead>
<tbody>
<tr><td>

**Bad**: Ghi log lỗi và trả về nó

Những người gọi ở các cấp cao hơn trong ngăn xếp có khả năng sẽ thực hiện hành động tương tự với lỗi.
Làm như vậy sẽ gây ra nhiều nhiễu trong nhật ký ứng dụng mà không có giá trị nhiều.

</td><td>

```go
u, err := getUser(id)
if err != nil {
  // BAD: Xem mô tả
  log.Printf("Could not get user %q: %v", id, err)
  return err
}
```

</td></tr>
<tr><td>

**Good**: Bọc lỗi và trả về nó

Những người gọi ở các cấp cao hơn trong ngăn xếp sẽ xử lý lỗi.
Sử dụng `%w` đảm bảo rằng họ có thể khớp lỗi với `errors.Is` hoặc `errors.As`
nếu cần thiết.

</td><td>

```go
u, err := getUser(id)
if err != nil {
  return fmt.Errorf("get user %q: %w", id, err)
}
```

</td></tr>
<tr><td>

**Good**: Ghi log lỗi và xử lý nhẹ nhàng

Nếu thao tác không thực sự cần thiết,
chúng ta có thể cung cấp một trải nghiệm bị suy giảm nhưng không bị gián đoạn
bằng cách khôi phục từ lỗi đó.

</td><td>

```go
if err := emitMetrics(); err != nil {
// Thất bại khi ghi số liệu không nên
// làm hỏng ứng dụng.
  log.Printf("Could not emit metrics: %v", err)
}

```

</td></tr>
<tr><td>

**Good**: Khớp lỗi và xử lý nhẹ nhàng

Nếu hàm được gọi định nghĩa một lỗi cụ thể trong hợp đồng của nó,
và lỗi có thể khôi phục,
hãy khớp lỗi đó và xử lý nhẹ nhàng.
Đối với tất cả các trường hợp khác, bọc lỗi và trả về nó.

Những người gọi ở các cấp cao hơn trong ngăn xếp sẽ xử lý các lỗi khác.

</td><td>

```go
tz, err := getUserTimeZone(id)
if err != nil {
  if errors.Is(err, ErrUserNotFound) {
    // Người dùng không tồn tại. Sử dụng UTC.
    tz = time.UTC
  } else {
    return fmt.Errorf("get user %q: %w", id, err)
  }
}
```

</td></tr>
</tbody></table>

### Handle Type Assertion Failures

Giá trị trả về đơn của một [type assertion](https://go.dev/ref/spec#Type_assertions) sẽ gây hoảng loạn (panic) khi gặp loại không đúng. Vì vậy, luôn sử dụng idiom "comma ok".

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // xử lý lỗi một cách nhẹ nhàng
}
```

</td></tr>
</tbody></table>

<!-- TODO: There are a few situations where the single assignment form is
fine. -->

### Don't Panic

Code chạy trong môi trường sản xuất phải tránh panics. Panics là một nguồn lớn gây ra [cascading failures](https://en.wikipedia.org/wiki/Cascading_failure). Nếu xảy ra lỗi, hàm phải trả về lỗi và cho phép hàm gọi quyết định cách xử lý.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
func run(args []string) {
  if len(args) == 0 {
    panic("an argument is required")
  }
  // ...
}

func main() {
  run(os.Args[1:])
}
```

</td><td>

```go
func run(args []string) error {
  if len(args) == 0 {
    return errors.New("an argument is required")
  }
  // ...
  return nil
}

func main() {
  if err := run(os.Args[1:]); err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
  }
}
```

</td></tr>
</tbody></table>

Panic/recover không phải là một chiến lược xử lý lỗi. Một chương trình chỉ nên panic khi có điều không thể phục hồi xảy ra như một trỏ đến nil. Một ngoại lệ cho quy tắc này là quá trình khởi tạo chương trình: các vấn đề xấu xảy ra khi bắt đầu chương trình mà nên dừng chương trình có thể gây ra panic.

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

Ngay cả trong các bài kiểm tra, ưu tiên sử dụng `t.Fatal` hoặc `t.FailNow` hơn panics để đảm bảo rằng bài kiểm tra được đánh dấu là thất bại.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>

### Use go.uber.org/atomic

Các hoạt động nguyên tử với gói [sync/atomic](https://pkg.go.dev/sync/atomic) hoạt động trên các loại dữ liệu nguyên thủy (`int32`, `int64`, vv.) nên dễ quên sử dụng hoạt động nguyên tử để đọc hoặc sửa đổi các biến.

[go.uber.org/atomic](https://pkg.go.dev/go.uber.org/atomic) thêm tính an toàn loại cho các hoạt động này bằng cách ẩn đi loại dữ liệu cơ bản. Ngoài ra, nó bao gồm một kiểu thuận tiện `atomic.Bool`.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
type foo struct {
  running int32  // atomic
}

func (f* foo) start() {
  if atomic.SwapInt32(&f.running, 1) == 1 {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running == 1  // race!
}
```

</td><td>

```go
type foo struct {
  running atomic.Bool
}

func (f *foo) start() {
  if f.running.Swap(true) {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running.Load()
}
```

</td></tr>
</tbody></table>

### Avoid Mutable Globals

Tránh thay đổi các biến toàn cục, thay vào đó lựa chọn tiêm phụ thuộc (dependency injection). Điều này áp dụng cho con trỏ hàm cũng như các loại giá trị khác.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
// sign.go

var _timeNow = time.Now

func sign(msg string) string {
  now := _timeNow()
  return signWithTime(msg, now)
}
```

</td><td>

```go
// sign.go

type signer struct {
  now func() time.Time
}

func newSigner() *signer {
  return &signer{
    now: time.Now,
  }
}

func (s *signer) Sign(msg string) string {
  now := s.now()
  return signWithTime(msg, now)
}
```

</td></tr>
<tr><td>

```go
// sign_test.go

func TestSign(t *testing.T) {
  oldTimeNow := _timeNow
  _timeNow = func() time.Time {
    return someFixedTime
  }
  defer func() { _timeNow = oldTimeNow }()

  assert.Equal(t, want, sign(give))
}
```

</td><td>

```go
// sign_test.go

func TestSigner(t *testing.T) {
  s := newSigner()
  s.now = func() time.Time {
    return someFixedTime
  }

  assert.Equal(t, want, s.Sign(give))
}
```

</td></tr>
</tbody></table>

### Avoid Embedding Types in Public Structs

Các loại nhúng này tiết lộ chi tiết cài đặt, ức chế sự tiến hóa loại và làm mờ tài liệu.

Giả sử bạn đã triển khai một loạt các loại danh sách bằng cách sử dụng một `AbstractList` chung, hãy tránh nhúng `AbstractList` trong các triển khai danh sách cụ thể của bạn. Thay vào đó, hãy viết thủ công chỉ các phương thức cho danh sách cụ thể của bạn mà sẽ ủy quyền cho danh sách trừu tượng.

```go
type AbstractList struct {}

// Add adds an entity to the list.
func (l *AbstractList) Add(e Entity) {
  // ...
}

// Remove removes an entity from the list.
func (l *AbstractList) Remove(e Entity) {
  // ...
}
```

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
// ConcreteList is a list of entities.
type ConcreteList struct {
  *AbstractList
}
```

</td><td>

```go
// ConcreteList is a list of entities.
type ConcreteList struct {
  list *AbstractList
}

// Add adds an entity to the list.
func (l *ConcreteList) Add(e Entity) {
  l.list.Add(e)
}

// Remove removes an entity from the list.
func (l *ConcreteList) Remove(e Entity) {
  l.list.Remove(e)
}
```

</td></tr>
</tbody></table>

Go cho phép [type embedding](https://go.dev/doc/effective_go#embedding) như một sự tha thứ giữa kế thừa và hợp thành. Loại bên ngoài nhận bản sao ngầm của các phương thức của loại được nhúng. Các phương thức này, mặc định, ủy quyền cho cùng một phương thức của thể hiện được nhúng.

Cấu trúc cũng nhận một trường có cùng tên với loại. Vì vậy, nếu loại được nhúng là công khai, trường cũng là công khai. Để duy trì tính tương thích ngược, mọi phiên bản tương lai của loại bên ngoài phải giữ loại được nhúng.

Một loại được nhúng hiếm khi cần thiết. Đây là một tiện ích giúp bạn tránh việc viết các phương thức ủy quyền tẻ nhạt.

Ngay cả việc nhúng một AbstractList _interface_ tương thích, thay vì cấu trúc, cũng sẽ cung cấp cho người phát triển linh hoạt hơn cho tương lai, nhưng vẫn tiết lộ chi tiết rằng các danh sách cụ thể sử dụng một triển khai trừu tượng.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
// AbstractList is a generalized implementation
// for various kinds of lists of entities.
type AbstractList interface {
  Add(Entity)
  Remove(Entity)
}

// ConcreteList is a list of entities.
type ConcreteList struct {
  AbstractList
}
```

</td><td>

```go
// ConcreteList is a list of entities.
type ConcreteList struct {
  list AbstractList
}

// Add adds an entity to the list.
func (l *ConcreteList) Add(e Entity) {
  l.list.Add(e)
}

// Remove removes an entity from the list.
func (l *ConcreteList) Remove(e Entity) {
  l.list.Remove(e)
}
```

</td></tr>
</tbody></table>

Dù là với một cấu trúc nhúng hoặc một giao diện nhúng, loại được nhúng đặt ra các giới hạn về sự phát triển của loại.

- Thêm các phương thức vào một giao diện được nhúng là một thay đổi gây hỏng.
- Loại bỏ các phương thức từ một cấu trúc được nhúng là một thay đổi gây hỏng.
- Loại bỏ loại được nhúng là một thay đổi gây hỏng.
- Thay thế loại được nhúng, ngay cả với một tùy chọn khác thỏa mãn cùng một giao diện, là một thay đổi gây hỏng.

Mặc dù việc viết các phương thức ủy quyền này là một công việc tẻ nhạt, nhưng sự nỗ lực bổ sung ẩn đi một chi tiết triển khai, tạo ra nhiều cơ hội cho sự thay đổi, và cũng loại bỏ sự trung gian để khám phá giao diện List đầy đủ trong tài liệu.

### Avoid Using Built-In Names

Theo [language specification](https://go.dev/ref/spec) của Go, có một số,
[predeclared identifiers](https://go.dev/ref/spec#Predeclared_identifiers) không nên được sử dụng như tên trong các chương trình Go.

Tùy thuộc vào ngữ cảnh, việc sử dụng lại các tên này như tên sẽ gây ra việc ẩn đi tên ban đầu trong phạm vi từ vựng hiện tại (và bất kỳ phạm vi lồng nhau nào) hoặc làm cho mã ảnh hưởng trở nên rối rắm. Trong trường hợp tốt nhất, trình biên dịch sẽ phàn nàn; trong trường hợp tồi nhất, mã như vậy có thể giới thiệu các lỗi ẩn khó tìm kiếm.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
var error string
// `error` shadows the builtin

// or

func handleErrorMessage(error string) {
    // `error` shadows the builtin
}
```

</td><td>

```go
var errorMessage string
// `error` refers to the builtin

// or

func handleErrorMessage(msg string) {
    // `error` refers to the builtin
}
```

</td></tr>
<tr><td>

```go
type Foo struct {
    // While these fields technically don't
    // constitute shadowing, grepping for
    // `error` or `string` strings is now
    // ambiguous.
    error  error
    string string
}

func (f Foo) Error() error {
    // `error` and `f.error` are
    // visually similar
    return f.error
}

func (f Foo) String() string {
    // `string` and `f.string` are
    // visually similar
    return f.string
}
```

</td><td>

```go
type Foo struct {
    // `error` and `string` strings are
    // now unambiguous.
    err error
    str string
}

func (f Foo) Error() error {
    return f.err
}

func (f Foo) String() string {
    return f.str
}
```

</td></tr>
</tbody></table>

Lưu ý rằng trình biên dịch sẽ không tạo ra lỗi khi sử dụng các từ ngữ được định nghĩa trước, nhưng các công cụ như `go vet` nên chỉ ra những trường hợp này cũng như các trường hợp khác của sự ẩn giấu.

### Avoid `init()`

Tránh sử dụng `init()` nếu có thể. Khi không thể tránh hoặc mong muốn sử dụng `init()`, mã nên cố gắng:

1. Hoàn toàn xác định, bất kể môi trường chương trình hoặc lời gọi.
2. Tránh phụ thuộc vào thứ tự hoặc ảnh hưởng của các hàm `init()` khác. Mặc dù thứ tự `init()` là rất được biết đến, mã có thể thay đổi, và do đó mối quan hệ giữa các hàm `init()` có thể làm cho mã trở nên dễ vỡ và dễ gây lỗi.
3. Tránh truy cập hoặc thay đổi trạng thái toàn cục hoặc môi trường, như thông tin máy tính, biến môi trường, thư mục làm việc, đối số/chương trình đầu vào của chương trình, v.v.
4. Tránh I/O, bao gồm cả việc tệp hệ thống tệp, mạng và các cuộc gọi hệ thống.

Mã không thể đáp ứng được những yêu cầu này có thể thuộc về dạng helper để được gọi như một phần của `main()` (hoặc nơi khác trong vòng đời của chương trình), hoặc được viết như là một phần của `main()` chính. Đặc biệt, các thư viện dự định được sử dụng bởi các chương trình khác nên chú ý đặc biệt để hoàn toàn xác định và không thực hiện "init magic".

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
type Foo struct {
    // ...
}

var _defaultFoo Foo

func init() {
    _defaultFoo = Foo{
        // ...
    }
}
```

</td><td>

```go
var _defaultFoo = Foo{
    // ...
}

// or, better, for testability:

var _defaultFoo = defaultFoo()

func defaultFoo() Foo {
    return Foo{
        // ...
    }
}
```

</td></tr>
<tr><td>

```go
type Config struct {
    // ...
}

var _config Config

func init() {
    // Bad: based on current directory
    cwd, _ := os.Getwd()

    // Bad: I/O
    raw, _ := os.ReadFile(
        path.Join(cwd, "config", "config.yaml"),
    )

    yaml.Unmarshal(raw, &_config)
}
```

</td><td>

```go
type Config struct {
    // ...
}

func loadConfig() Config {
    cwd, err := os.Getwd()
    // handle err

    raw, err := os.ReadFile(
        path.Join(cwd, "config", "config.yaml"),
    )
    // handle err

    var config Config
    yaml.Unmarshal(raw, &config)

    return config
}
```

</td></tr>
</tbody></table>

Xem xét những trường hợp sau đây, trong đó `init()` có thể được ưu tiên hoặc
cần thiết:

- Các biểu thức phức tạp không thể được biểu diễn dưới dạng các phép gán đơn.
- Các hook có thể được cắm vào, như các ngôn ngữ `database/sql`, các đăng ký loại mã hóa, vv.
- Tối ưu hóa cho [Google Cloud Functions](https://cloud.google.com/functions/docs/bestpractices/tips#use_global_variables_to_reuse_objects_in_future_invocations) và các dạng tính toán trước xác định khác.

### Exit in Main

Chương trình Go sử dụng [`os.Exit`](https://pkg.go.dev/os#Exit) hoặc [`log.Fatal*`](https://pkg.go.dev/log#Fatal) để thoát ngay lập tức. (Chấm dứt bằng cách gây ra sự cố không phải là cách tốt để thoát chương trình, vui lòng [don't panic](#dont-panic).)

Gọi một trong số `os.Exit` hoặc `log.Fatal*` **chỉ trong `main()`**. Tất cả các hàm khác
nên trả về lỗi để báo hiệu thất bại.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
func main() {
  body := readFile(path)
  fmt.Println(body)
}

func readFile(path string) string {
  f, err := os.Open(path)
  if err != nil {
    log.Fatal(err)
  }

  b, err := io.ReadAll(f)
  if err != nil {
    log.Fatal(err)
  }

  return string(b)
}
```

</td><td>

```go
func main() {
  body, err := readFile(path)
  if err != nil {
    log.Fatal(err)
  }
  fmt.Println(body)
}

func readFile(path string) (string, error) {
  f, err := os.Open(path)
  if err != nil {
    return "", err
  }

  b, err := io.ReadAll(f)
  if err != nil {
    return "", err
  }

  return string(b), nil
}
```

</td></tr>
</tbody></table>

Lý do: Các chương trình có nhiều hàm thoát gặp một số vấn đề:

- Luồng điều khiển không rõ ràng: Bất kỳ hàm nào cũng có thể thoát khỏi chương trình nên trở nên khó hiểu về luồng điều khiển.
- Khó thử nghiệm: Một hàm thoát khỏi chương trình cũng sẽ thoát khỏi bài kiểm tra gọi nó. Điều này làm cho việc kiểm thử hàm trở nên khó khăn và tạo ra rủi ro bỏ qua các bài kiểm tra khác mà chưa được chạy bởi `go test`.
- Bỏ qua việc dọn dẹp: Khi một hàm thoát khỏi chương trình, nó sẽ bỏ qua các cuộc gọi hàm đã được đặt hàng với các lệnh `defer`. Điều này tạo ra nguy cơ bỏ qua các nhiệm vụ dọn dẹp quan trọng.

#### Exit Once

Nếu có thể, ưu tiên gọi `os.Exit` hoặc `log.Fatal` **tối đa một lần** trong hàm `main()` của bạn. Nếu có nhiều kịch bản lỗi dừng thực thi chương trình, hãy đặt logic đó trong một hàm riêng và trả về lỗi từ đó.

Điều này có tác dụng rút ngắn hàm `main()` của bạn và đặt tất cả các logic kinh doanh chính vào một hàm riêng, có thể kiểm thử được.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
package main

func main() {
  args := os.Args[1:]
  if len(args) != 1 {
    log.Fatal("missing file")
  }
  name := args[0]

  f, err := os.Open(name)
  if err != nil {
    log.Fatal(err)
  }
  defer f.Close()

  // If we call log.Fatal after this line,
  // f.Close will not be called.

  b, err := io.ReadAll(f)
  if err != nil {
    log.Fatal(err)
  }

  // ...
}
```

</td><td>

```go
package main

func main() {
  if err := run(); err != nil {
    log.Fatal(err)
  }
}

func run() error {
  args := os.Args[1:]
  if len(args) != 1 {
    return errors.New("missing file")
  }
  name := args[0]

  f, err := os.Open(name)
  if err != nil {
    return err
  }
  defer f.Close()

  b, err := io.ReadAll(f)
  if err != nil {
    return err
  }

  // ...
}
```

</td></tr>
</tbody></table>

Ví dụ trên sử dụng `log.Fatal`, nhưng hướng dẫn cũng áp dụng cho
`os.Exit` hoặc bất kỳ mã thư viện nào gọi `os.Exit`.

```go
func main() {
  if err := run(); err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
  }
}
```

Bạn có thể thay đổi chữ ký của `run()` để phù hợp với nhu cầu của bạn.
Ví dụ, nếu chương trình của bạn phải thoát với các mã thoát cụ thể cho các lỗi,
`run()` có thể trả về mã thoát thay vì một lỗi.
Điều này cho phép các bài kiểm thử đơn vị kiểm tra trực tiếp hành vi này cũng.

```go
func main() {
  os.Exit(run(args))
}

func run() (exitCode int) {
  // ...
}
```

Nói một cách tổng quát, lưu ý rằng hàm `run()` được sử dụng trong các ví dụ này
không có ý định hướng dẫn cụ thể.
Có sự linh hoạt trong tên, chữ ký và cài đặt của hàm `run()`.
Ngoài những điều khác, bạn có thể:

- chấp nhận các đối số dòng lệnh chưa được phân tích (ví dụ: `run(os.Args[1:])`)
- phân tích các đối số dòng lệnh trong `main()` và chuyển chúng vào `run`
- sử dụng một loại lỗi tùy chỉnh để mang mã thoát trở lại `main()`
- đặt `business logic` trong một lớp trừu tượng khác với `package main`

Hướng dẫn này chỉ yêu cầu có một nơi duy nhất trong hàm `main()` của bạn
chịu trách nhiệm thực sự thoát quá trình.

### Use field tags in marshaled structs

Bất kỳ trường struct nào được mã hóa thành JSON, YAML,
hoặc các định dạng khác hỗ trợ đặt tên trường dựa trên thẻ
nên được chú thích bằng thẻ liên quan.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
type Stock struct {
  Price int
  Name  string
}

bytes, err := json.Marshal(Stock{
  Price: 137,
  Name:  "UBER",
})
```

</td><td>

```go
type Stock struct {
  Price int    `json:"price"`
  Name  string `json:"name"`
  // Safe to rename Name to Symbol.
}

bytes, err := json.Marshal(Stock{
  Price: 137,
  Name:  "UBER",
})
```

</td></tr>
</tbody></table>

Lý do:
Hình thức được chuỗi hóa của cấu trúc là một hợp đồng giữa các hệ thống khác nhau.
Các thay đổi vào cấu trúc của hình thức được chuỗi hóa - bao gồm tên trường - sẽ phá vỡ
hợp đồng này. Chỉ định tên trường bên trong các thẻ làm cho hợp đồng trở nên rõ ràng,
và nó ngăn chặn việc phá vỡ hợp đồng một cách tình cờ thông qua việc tái cấu trúc hoặc
đổi tên trường.

### Don't fire-and-forget goroutines

Goroutines là nhẹ nhàng, nhưng chúng không phải là miễn phí:
ít nhất, chúng tạo ra chi phí về bộ nhớ cho ngăn xếp của chúng và CPU để được lên lịch.
Mặc dù những chi phí này nhỏ đối với việc sử dụng goroutines điển hình,
chúng có thể gây ra vấn đề hiệu suất đáng kể
khi được khởi tạo trong số lượng lớn mà không kiểm soát được thời gian tồn tại.
Các goroutines với thời gian tồn tại không được quản lý cũng có thể gây ra các vấn đề khác
như ngăn chặn các đối tượng không được sử dụng khỏi việc thu gom rác
và giữ các tài nguyên mà về cơ bản không còn được sử dụng nữa.

Do đó, không rò rỉ goroutines trong mã sản xuất.
Sử dụng [go.uber.org/goleak](https://pkg.go.dev/go.uber.org/goleak)
để kiểm tra rò rỉ goroutine trong các gói mà có thể tạo ra goroutines.

Nói chung, mỗi goroutine:

- phải có một thời gian dừng dự đoán được khi nó sẽ dừng chạy; hoặc
- phải có một cách để gửi tín hiệu cho goroutine biết rằng nó nên dừng lại

Trong cả hai trường hợp, phải có một cách mã để chặn và đợi goroutine
hoàn thành.

Ví dụ:

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
go func() {
  for {
    flush()
    time.Sleep(delay)
  }
}()
```

</td><td>

```go
var (
  stop = make(chan struct{}) // tells the goroutine to stop
  done = make(chan struct{}) // tells us that the goroutine exited
)
go func() {
  defer close(done)

  ticker := time.NewTicker(delay)
  defer ticker.Stop()
  for {
    select {
    case <-ticker.C:
      flush()
    case <-stop:
      return
    }
  }
}()

// Elsewhere...
close(stop)  // signal the goroutine to stop
<-done       // and wait for it to exit
```

</td></tr>
<tr><td>

Không có cách nào để dừng goroutine này.
Nó sẽ chạy cho đến khi ứng dụng thoát.

</td><td>

Goroutine này có thể được dừng lại bằng cách `close(stop)`,
và chúng ta có thể chờ nó thoát với `<-done`.

</td></tr>
</tbody></table>

#### Wait for goroutines to exit

Cho một goroutine được khởi tạo bởi hệ thống,
phải có một cách để chờ goroutine thoát.
Có hai cách phổ biến để làm điều này:

- Sử dụng `sync.WaitGroup`.
  Làm như vậy nếu có nhiều goroutine bạn muốn chờ đợi

  ```go
  var wg sync.WaitGroup
  for i := 0; i < N; i++ {
    wg.Add(1)
    go func() {
      defer wg.Done()
      // ...
    }()
  }

  // Để chờ tất cả kết thúc:
  wg.Wait()
  ```

- Thêm một `chan struct{}` khác mà goroutine đó đóng khi đã hoàn thành.
  Làm như vậy nếu chỉ có một goroutine.

  ```go
  done := make(chan struct{})
  go func() {
    defer close(done)
    // ...
  }()

  // To wait for the goroutine to finish:
  <-done
  ```

#### No goroutines in `init()`

Các hàm `init()` không nên khởi tạo goroutine.
Xem thêm [Avoid init()](#avoid-init).

Nếu một gói cần một goroutine nền,
nó phải tiếp tục một đối tượng có trách nhiệm quản lý thời gian sống của một goroutine.
Đối tượng phải cung cấp một phương thức (`Close`, `Stop`, `Shutdown`, vv)
để gửi tín hiệu cho goroutine nền để dừng lại và chờ cho nó thoát.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
func init() {
  go doWork()
}

func doWork() {
  for {
    // ...
  }
}
```

</td><td>

```go
type Worker struct{ /* ... */ }

func NewWorker(...) *Worker {
  w := &Worker{
    stop: make(chan struct{}),
    done: make(chan struct{}),
    // ...
  }
  go w.doWork()
}

func (w *Worker) doWork() {
  defer close(w.done)
  for {
    // ...
    case <-w.stop:
      return
  }
}

// Shutdown tells the worker to stop
// and waits until it has finished.
func (w *Worker) Shutdown() {
  close(w.stop)
  <-w.done
}
```

</td></tr>
<tr><td>

Khởi tạo một goroutine nền mà không kiểm soát khi người dùng xuất gói này.
Người dùng không có kiểm soát nào đối với goroutine hoặc một cách để dừng nó.

</td><td>

Khởi tạo người thực thi chỉ khi người dùng yêu cầu.
Cung cấp một cách để tắt người thực thi để người dùng có thể giải phóng
tài nguyên được sử dụng bởi người thực thi.

Lưu ý rằng bạn nên sử dụng `WaitGroup` nếu người thực thi quản lý nhiều
goroutines.
Xem [Wait for goroutines to exit](#wait-for-goroutines-to-exit).

</td></tr>
</tbody></table>

## Performance

Hướng dẫn cụ thể về hiệu suất chỉ áp dụng cho lối đi nhanh.

### Prefer strconv over fmt

Khi chuyển đổi các nguyên thủy thành/ từ chuỗi, `strconv` nhanh hơn
`fmt`.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}
```

</td><td>

```go
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}
```

</td></tr>
<tr><td>

```plain
BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
```

</td><td>

```plain
BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
```

</td></tr>
</tbody></table>

### Avoid repeated string-to-byte conversions

Không tạo byte slices từ một chuỗi cố định một cách lặp đi lặp lại. Thay vào đó, thực hiện việc chuyển đổi một lần và lưu kết quả.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}
```

</td><td>

```go
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}
```

</td></tr>
<tr><td>

```plain
BenchmarkBad-4   50000000   22.2 ns/op
```

</td><td>

```plain
BenchmarkGood-4  500000000   3.25 ns/op
```

</td></tr>
</tbody></table>

### Prefer Specifying Container Capacity

Chỉ định dung lượng của các container khi có thể để phân bổ bộ nhớ cho container ngay từ đầu. Điều này giảm thiểu các phân bổ sau này (bằng cách sao chép và thay đổi kích thước của container) khi các phần tử được thêm vào.

#### Specifying Map Capacity Hints

Khi có thể, cung cấp gợi ý về dung lượng khi khởi tạo map bằng `make()`.

```go
make(map[T1]T2, hint)
```

Việc cung cấp gợi ý về dung lượng cho `make()` cố gắng đặt kích thước của map vừa phải ngay từ thời điểm khởi tạo, điều này giảm thiểu nhu cầu mở rộng map và các phân bổ khi các phần tử được thêm vào map.

Lưu ý rằng, khác với slices, gợi ý về dung lượng của map không đảm bảo phân bổ hoàn toàn, tiên đoán, nhưng được sử dụng để ước lượng số lượng các bucket của hashmap cần thiết. Do đó, các phân bổ vẫn có thể xảy ra khi thêm phần tử vào map, thậm chí lên đến dung lượng được chỉ định.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[string]os.FileInfo)

files, _ := os.ReadDir("./files")
for _, f := range files {
    m[f.Name()] = f
}
```

</td><td>

```go

files, _ := os.ReadDir("./files")

m := make(map[string]os.DirEntry, len(files))
for _, f := range files {
    m[f.Name()] = f
}
```

</td></tr>
<tr><td>

`m` được tạo mà không có gợi ý về kích thước; có thể có nhiều phân bổ hơn khi gán.

</td><td>

`m` được tạo với một gợi ý về kích thước; có thể có ít phân bổ hơn khi gán.

</td></tr>
</tbody></table>

#### Specifying Slice Capacity

Khi có thể, cung cấp gợi ý về dung lượng khi khởi tạo slices bằng `make()`,
đặc biệt là khi thêm vào.

```go
make([]T, length, capacity)
```

Khác với maps, dung lượng của slice không phải là một gợi ý: trình biên dịch sẽ phân bổ đủ bộ nhớ cho dung lượng của slice như được cung cấp cho `make()`, điều này có nghĩa là các thao tác `append()` sau đó sẽ không gây ra bất kỳ phân bổ nào (cho đến khi độ dài của slice khớp với dung lượng, sau đó bất kỳ `append()` nào cũng sẽ đòi hỏi một phép thay đổi kích thước để chứa các phần tử bổ sung).

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
for n := 0; n < b.N; n++ {
  data := make([]int, 0)
  for k := 0; k < size; k++{
    data = append(data, k)
  }
}
```

</td><td>

```go
for n := 0; n < b.N; n++ {
  data := make([]int, 0, size)
  for k := 0; k < size; k++{
    data = append(data, k)
  }
}
```

</td></tr>
<tr><td>

```plain
BenchmarkBad-4    100000000    2.48s
```

</td><td>

```plain
BenchmarkGood-4   100000000    0.21s
```

</td></tr>
</tbody></table>

## Style

### Avoid overly long lines

Tránh các dòng mã yêu cầu người đọc cuộn ngang hoặc nghiêng đầu quá nhiều.

Chúng tôi khuyến nghị một giới hạn độ dài dòng mềm là **99 ký tự**.
Tác giả nên cố gắng gói dòng trước khi đạt đến giới hạn này,
nhưng đây không phải là một giới hạn cứng.
Mã được phép vượt qua giới hạn này.

### Be Consistent

Một số hướng dẫn được trình bày trong tài liệu này có thể được đánh giá một cách khách quan;
một số khác là tình huống, ngữ cảnh hoặc chủ quan.

Trên hết, **be consistent**.

Mã đồng nhất dễ bảo trì hơn, dễ hợp lý hóa hơn, đòi hỏi ít công sức nhận thức hơn và dễ di chuyển hoặc cập nhật khi các quy ước mới xuất hiện hoặc các lớp lỗi được sửa chữa.

Ngược lại, việc có nhiều phong cách không liên quan hoặc xung đột trong một codebase duy nhất gây ra chi phí bảo trì, sự không chắc chắn và sự không nhất quán nhận thức, tất cả đều có thể góp phần trực tiếp vào tốc độ thấp hơn, việc đánh giá mã đau đớn và lỗi.

Khi áp dụng các hướng dẫn này vào một codebase, khuyến nghị là các thay đổi được thực hiện ở mức gói (hoặc lớn hơn): việc áp dụng ở mức gói con vi phạm vấn đề trên bằng cách giới thiệu nhiều phong cách vào cùng một mã.

### Group Similar Declarations

Go hỗ trợ nhóm các khai báo tương tự.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
import "a"
import "b"
```

</td><td>

```go
import (
  "a"
  "b"
)
```

</td></tr>
</tbody></table>

Điều này cũng áp dụng cho các hằng số, biến và khai báo kiểu.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go

const a = 1
const b = 2



var a = 1
var b = 2



type Area float64
type Volume float64
```

</td><td>

```go
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)
```

</td></tr>
</tbody></table>

Chỉ nhóm các khai báo liên quan. Đừng nhóm các khai báo không liên quan.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
  EnvVar = "MY_ENV"
)
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

const EnvVar = "MY_ENV"
```

</td></tr>
</tbody></table>

Các nhóm không bị giới hạn về nơi chúng có thể được sử dụng. Ví dụ, bạn có thể sử dụng chúng
bên trong các hàm.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
func f() string {
  red := color.New(0xff0000)
  green := color.New(0x00ff00)
  blue := color.New(0x0000ff)

  // ...
}
```

</td><td>

```go
func f() string {
  var (
    red   = color.New(0xff0000)
    green = color.New(0x00ff00)
    blue  = color.New(0x0000ff)
  )

  // ...
}
```

</td></tr>
</tbody></table>

Exception: Các khai báo biến, đặc biệt là bên trong các hàm, nên được nhóm lại nếu được khai báo kề nhau với các biến khác. Làm điều này cho các biến được khai báo cùng nhau ngay cả khi chúng không liên quan.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
func (c *client) request() {
  caller := c.name
  format := "json"
  timeout := 5*time.Second
  var err error

  // ...
}
```

</td><td>

```go
func (c *client) request() {
  var (
    caller  = c.name
    format  = "json"
    timeout = 5*time.Second
    err error
  )

  // ...
}
```

</td></tr>
</tbody></table>

### Import Group Ordering

Nên có hai nhóm import:

- Thư viện chuẩn
- Tất cả các thứ khác

Đây là cách nhóm được áp dụng mặc định bởi goimports.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>

### Package Names

Khi đặt tên gói, chọn một tên như sau:

- Tất cả chữ thường. Không viết hoa hoặc dấu gạch dưới.
- Không cần phải đổi tên bằng cách sử dụng import có tên tại hầu hết các điểm gọi.
- Ngắn gọn và súc tích. Hãy nhớ rằng tên được nhận dạng đầy đủ tại mỗi điểm gọi.
- Không phải là số nhiều. Ví dụ, `net/url`, không phải `net/urls`.
- Không phải là "common", "util", "shared", hoặc "lib". Đây là các tên tệ, không cung cấp thông tin.

Xem thêm [Package Names](https://go.dev/blog/package-names) và [Style guideline for Go packages](https://rakyll.org/style-packages/).

### Function Names

Chúng tôi tuân theo quy ước của cộng đồng Go bằng cách sử dụng [MixedCaps for function
names](https://go.dev/doc/effective_go#mixed-caps). Một ngoại lệ được tạo ra cho các hàm kiểm tra, có thể chứa dấu gạch dưới
để nhóm các trường hợp kiểm tra liên quan, ví dụ,
`TestMyFunction_WhatIsBeingTested`.

### Import Aliasing

Đặt tên định danh import phải được sử dụng nếu tên gói không khớp với phần cuối của đường dẫn import.

```go
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```

Trong tất cả các tình huống khác, việc tránh sử dụng định danh import nên được thực hiện trừ khi có xung đột trực tiếp giữa các import.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"


  nettrace "golang.net/x/trace"
)
```

</td><td>

```go
import (
  "fmt"
  "os"
  "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

</td></tr>
</tbody></table>

### Function Grouping and Ordering

- Các hàm nên được sắp xếp theo thứ tự gọi gần đúng.
- Các hàm trong một tệp nên được nhóm theo receiver.

Do đó, các hàm được xuất nên xuất hiện đầu tiên trong một tệp, sau các định nghĩa `struct`, `const`, `var`.

Một `newXYZ()`/`NewXYZ()` có thể xuất hiện sau khi kiểu được định nghĩa, nhưng trước phần còn lại của các phương thức trên receiver.

Vì các hàm được nhóm theo receiver, các hàm tiện ích đơn giản nên xuất hiện ở cuối tệp.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>

### Reduce Nesting

Mã nên giảm thiểu sự lồng nhau nơi có thể bằng cách xử lý các trường hợp lỗi/điều kiện đặc biệt trước và trả về sớm hoặc tiếp tục vòng lặp. Giảm số lượng mã được lồng nhiều cấp độ.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    if err := v.Call(); err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

</td><td>

```go
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  if err := v.Call(); err != nil {
    return err
  }
  v.Send()
}
```

</td></tr>
</tbody></table>

### Unnecessary Else

Nếu một biến được thiết lập trong cả hai nhánh của một câu lệnh if, nó có thể được thay thế bằng một câu lệnh if duy nhất.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
var a int
if b {
  a = 100
} else {
  a = 10
}
```

</td><td>

```go
a := 10
if b {
  a = 100
}
```

</td></tr>
</tbody></table>

### Top-level Variable Declarations

Ở cấp độ cao nhất, sử dụng từ khóa tiêu chuẩn `var`. Không chỉ định kiểu, trừ khi nó không phải là cùng kiểu với biểu thức.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Since F already states that it returns a string, we don't need to specify
// the type again.

func F() string { return "A" }
```

</td></tr>
</tbody></table>

Chỉ định kiểu nếu kiểu của biểu thức không khớp chính xác với kiểu mong muốn.

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F returns an object of type myError but we want error.
```

### Prefix Unexported Globals with \_

Thêm tiền tố `_` cho các `var` và `const` cấp độ cao không được xuất để làm cho rõ ràng khi chúng được sử dụng rằng chúng là các ký hiệu toàn cục.

Lý do: Các biến và hằng số cấp độ cao có phạm vi gói. Sử dụng một tên thông thường làm cho việc sử dụng giá trị sai sót dễ dàng xảy ra trong một tệp khác.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
// foo.go

const (
  defaultPort = 8080
  defaultUser = "user"
)

// bar.go

func Bar() {
  defaultPort := 9090
  ...
  fmt.Println("Default port", defaultPort)

  // We will not see a compile error if the first line of
  // Bar() is deleted.
}
```

</td><td>

```go
// foo.go

const (
  _defaultPort = 8080
  _defaultUser = "user"
)
```

</td></tr>
</tbody></table>

**Exception**: Các giá trị lỗi không được xuất có thể sử dụng tiền tố `err` mà không có gạch dưới.
Xem [Error Naming](#error-naming).

### Embedding in Structs

Các loại nhúng nên được đặt ở đầu danh sách trường của một
struct, và phải có một dòng trống phân tách các trường nhúng khỏi các trường thông thường.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
type Client struct {
  version int
  http.Client
}
```

</td><td>

```go
type Client struct {
  http.Client

  version int
}
```

</td></tr>
</tbody></table>

Việc nhúng phải cung cấp lợi ích cụ thể, như thêm hoặc mở rộng
chức năng theo cách phù hợp với ngữ cảnh ý nghĩa. Nó phải làm điều này mà không có
tác động tiêu cực nào đối với người dùng (xem thêm: [Avoid Embedding Types in Public Structs](#avoid-embedding-types-in-public-structs)).

Ngoại lệ: Mutexes không nên được nhúng, ngay cả trên các loại không được xuất. Xem thêm: [Zero-value Mutexes are Valid](#zero-value-mutexes-are-valid).

Việc nhúng **không nên**:

- Chỉ mang tính trang trí hoặc thuận tiện.
- Làm cho các loại bên ngoài khó xây dựng hoặc sử dụng hơn.
- Ảnh hưởng đến giá trị zero của các loại bên ngoài. Nếu loại bên ngoài có một giá trị zero hữu ích, nó
  vẫn nên có một giá trị zero hữu ích sau khi nhúng loại bên trong.
- Tiết lộ các hàm hoặc trường không liên quan từ loại bên ngoài như là một hiệu ứng phụ của việc
  nhúng loại bên trong.
- Tiết lộ các loại không được xuất.
- Ảnh hưởng đến cách sao chép của các loại bên ngoài.
- Thay đổi API hoặc ý nghĩa loại của các loại bên ngoài.
- Nhúng một dạng không chuẩn của loại bên trong.
- Tiết lộ chi tiết cài đặt của loại bên ngoài.
- Cho phép người dùng quan sát hoặc điều khiển các bộ phận bên trong loại.
- Thay đổi hành vi tổng quát của các hàm bên trong thông qua việc bọc một cách
  mà có thể làm người dùng ngạc nhiên một cách hợp lý.

Đơn giản là, nhúng một cách có ý thức và cố ý. Một thử nghiệm tốt là, "tất cả các phương thức/trường bên trong được xuất này có được thêm trực tiếp vào loại bên ngoài không"; nếu câu trả lời là "một số" hoặc "không", đừng nhúng loại bên trong - hãy sử dụng một trường thay vào đó.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
type A struct {
    // Bad: A.Lock() and A.Unlock() are
    //      now available, provide no
    //      functional benefit, and allow
    //      users to control details about
    //      the internals of A.
    sync.Mutex
}
```

</td><td>

```go
type countingWriteCloser struct {
    // Good: Write() is provided at this
    //       outer layer for a specific
    //       purpose, and delegates work
    //       to the inner type's Write().
    io.WriteCloser

    count int
}

func (w *countingWriteCloser) Write(bs []byte) (int, error) {
    w.count += len(bs)
    return w.WriteCloser.Write(bs)
}
```

</td></tr>
<tr><td>

```go
type Book struct {
    // Bad: pointer changes zero value usefulness
    io.ReadWriter

    // other fields
}

// later

var b Book
b.Read(...)  // panic: nil pointer
b.String()   // panic: nil pointer
b.Write(...) // panic: nil pointer
```

</td><td>

```go
type Book struct {
    // Good: has useful zero value
    bytes.Buffer

    // other fields
}

// later

var b Book
b.Read(...)  // ok
b.String()   // ok
b.Write(...) // ok
```

</td></tr>
<tr><td>

```go
type Client struct {
    sync.Mutex
    sync.WaitGroup
    bytes.Buffer
    url.URL
}
```

</td><td>

```go
type Client struct {
    mtx sync.Mutex
    wg  sync.WaitGroup
    buf bytes.Buffer
    url url.URL
}
```

</td></tr>
</tbody></table>

### Local Variable Declarations

Khai báo biến ngắn gọn (`:=`) nên được sử dụng nếu một biến được thiết lập với một giá trị cụ thể.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
var s = "foo"
```

</td><td>

```go
s := "foo"
```

</td></tr>
</tbody></table>

Tuy nhiên, có những trường hợp mà giá trị mặc định rõ ràng hơn khi sử dụng từ khóa `var`. [Declaring Empty Slices](https://go.dev/wiki/CodeReviewComments#declaring-empty-slices), ví dụ.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td><td>

```go
func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td></tr>
</tbody></table>

### nil is a valid slice

`nil` là một slice hợp lệ với độ dài 0. Điều này có nghĩa là,

- Bạn không nên trả về một slice có độ dài bằng 0 một cách rõ ràng. Thay vào đó, hãy trả về `nil`.

  <table>
  <thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  if x == "" {
    return []int{}
  }
  ```

  </td><td>

  ```go
  if x == "" {
    return nil
  }
  ```

  </td></tr>
  </tbody></table>

- Để kiểm tra xem một slice có rỗng hay không, luôn sử dụng `len(s) == 0`. Đừng kiểm tra `nil`.

  <table>
  <thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  func isEmpty(s []string) bool {
    return s == nil
  }
  ```

  </td><td>

  ```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
  ```

  </td></tr>
  </tbody></table>

- Giá trị zero (một slice được khai báo với `var`) có thể sử dụng ngay lập tức mà không cần `make()`.

  <table>
  <thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  nums := []int{}
  // or, nums := make([]int)

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td><td>

  ```go
  var nums []int

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td></tr>
  </tbody></table>

Nhớ rằng, mặc dù nó là một slice hợp lệ, một slice nil không tương đương với một slice đã được cấp phát với độ dài 0 - một cái là nil và cái kia không - và hai cái có thể được xử lý khác nhau trong các tình huống khác nhau (như làm phẳng).

### Reduce Scope of Variables

Nếu có thể, hãy giảm phạm vi của biến. Đừng giảm phạm vi nếu nó xung đột với [Reduce Nesting](#reduce-nesting).

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
err := os.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

</td><td>

```go
if err := os.WriteFile(name, data, 0644); err != nil {
 return err
}
```

</td></tr>
</tbody></table>

Nếu bạn cần một kết quả của một cuộc gọi hàm bên ngoài câu lệnh if, thì bạn không nên cố gắng giảm phạm vi.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
if data, err := os.ReadFile(name); err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

</td><td>

```go
data, err := os.ReadFile(name)
if err != nil {
   return err
}

if err := cfg.Decode(data); err != nil {
  return err
}

fmt.Println(cfg)
return nil
```

</td></tr>
</tbody></table>

### Avoid Naked Parameters

Các tham số không có tên trong cuộc gọi hàm có thể làm giảm khả năng đọc hiểu. Thêm các comment theo kiểu C (`/* ... */`) cho tên tham số khi ý nghĩa của chúng không rõ ràng.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

Tốt hơn nữa, thay thế các kiểu `bool` không có tên với các kiểu tùy chỉnh để làm cho mã đọc hiểu và an toàn hơn về kiểu. Điều này cho phép nhiều hơn chỉ hai trạng thái (đúng/sai) cho tham số đó trong tương lai.

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady Status = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

### Use Raw String Literals to Avoid Escaping

Go hỗ trợ [raw string literals](https://go.dev/ref/spec#raw_string_lit),
có thể trải dài qua nhiều dòng và bao gồm dấu ngoặc kép. Sử dụng chúng để tránh các chuỗi đã được escape thủ công, mà khó đọc hơn nhiều.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>

### Initializing Structs

#### Use Field Names to Initialize Structs

Bạn nên gần như luôn chỉ định tên trường khi khởi tạo các cấu trúc. Điều này hiện được bắt buộc bởi [`go vet`](https://pkg.go.dev/cmd/vet).

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

Exception: Tên trường _có thể_ được bỏ qua trong các bảng kiểm tra khi có 3 hoặc ít hơn 3 trường.

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```

#### Omit Zero Value Fields in Structs

Khi khởi tạo cấu trúc với tên trường, bỏ qua các trường có giá trị không nếu chúng không cung cấp ngữ cảnh ý nghĩa. Nếu không, để cho Go tự động đặt chúng thành các giá trị không.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
user := User{
  FirstName: "John",
  LastName: "Doe",
  MiddleName: "",
  Admin: false,
}
```

</td><td>

```go
user := User{
  FirstName: "John",
  LastName: "Doe",
}
```

</td></tr>
</tbody></table>

Điều này giúp giảm tiếng ồn cho người đọc bằng cách bỏ qua các giá trị mặc định trong ngữ cảnh đó. Chỉ có các giá trị ý nghĩa được chỉ định.

Bao gồm các giá trị không khi tên trường cung cấp ngữ cảnh ý nghĩa. Ví dụ, các trường hợp kiểm tra trong [Test Tables](#test-tables) có thể hưởng lợi từ tên của các trường ngay cả khi chúng có giá trị không.

```go
tests := []struct{
  give string
  want int
}{
  {give: "0", want: 0},
  // ...
}
```

#### Use `var` for Zero Value Structs

Khi tất cả các trường của một cấu trúc được bỏ qua trong một khai báo, sử dụng hình thức `var` để khai báo cấu trúc.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
user := User{}
```

</td><td>

```go
var user User
```

</td></tr>
</tbody></table>

Điều này phân biệt các cấu trúc có giá trị không từ các cấu trúc có các trường không bằng không tương tự như phân biệt tạo ra cho [map initialization](#initializing-maps), và phù hợp với cách chúng tôi ưu tiên [declare empty slices](https://go.dev/wiki/CodeReviewComments#declaring-empty-slices).

#### Initializing Struct References

Sử dụng `&T{}` thay vì `new(T)` khi khởi tạo tham chiếu cấu trúc để đồng nhất với việc khởi tạo cấu trúc.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>

### Initializing Maps

Ưu tiên sử dụng `make(..)` cho các map rỗng và các map được điền dữ liệu theo chương trình. Điều này làm cho việc khởi tạo map khác biệt về mặt trực quan so với việc khai báo, và nó làm cho việc thêm gợi ý kích thước sau này dễ dàng hơn nếu có sẵn.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = map[T1]T2{}
  m2 map[T1]T2
)
```

</td><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = make(map[T1]T2)
  m2 map[T1]T2
)
```

</td></tr>
<tr><td>

DKhai báo và khởi tạo trực quan giống nhau.

</td><td>

Khai báo và khởi tạo trực quan khác biệt.

</td></tr>
</tbody></table>

Nếu có thể, cung cấp gợi ý về dung lượng khi khởi tạo map bằng cách sử dụng `make()`. Xem
[Specifying Map Capacity Hints](#specifying-map-capacity-hints)
để biết thêm thông tin.

Tuy nhiên, nếu map chứa một danh sách cố định các phần tử, hãy sử dụng các biểu thức map để khởi tạo map.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[T1]T2, 3)
m[k1] = v1
m[k2] = v2
m[k3] = v3
```

</td><td>

```go
m := map[T1]T2{
  k1: v1,
  k2: v2,
  k3: v3,
}
```

</td></tr>
</tbody></table>

Quy tắc cơ bản là sử dụng biểu thức map khi thêm một tập hợp cố định các phần tử vào thời điểm khởi tạo, nếu không thì sử dụng `make` (và chỉ định một gợi ý về kích thước nếu có sẵn).

### Format Strings outside Printf

Nếu bạn khai báo chuỗi định dạng cho các hàm kiểu `Printf` ngoài chuỗi ký tự, hãy làm cho chúng trở thành các giá trị `const`.

Điều này giúp `go vet` thực hiện phân tích tĩnh của chuỗi định dạng.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td><td>

```go
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td></tr>
</tbody></table>

### Naming Printf-style Functions

Khi khai báo một hàm kiểu `Printf`, hãy đảm bảo rằng `go vet` có thể phát hiện và kiểm tra chuỗi định dạng.

Điều này có nghĩa là bạn nên sử dụng các tên hàm kiểu `Printf` được định nghĩa trước nếu có thể. `go vet` sẽ kiểm tra những cái này mặc định. Xem [Printf family](https://pkg.go.dev/cmd/vet#hdr-Printf_family)
để biết thêm thông tin.

Nếu việc sử dụng các tên được định nghĩa trước không phải là một lựa chọn, kết thúc tên bạn chọn bằng f: `Wrapf`, không phải là `Wrap`. `go vet` có thể được yêu cầu kiểm tra các tên kiểu `Printf` cụ thể nhưng chúng phải kết thúc bằng f.

```shell
go vet -printfuncs=wrapf,statusf
```

See also [go vet: Printf family check](https://kuzminva.wordpress.com/2017/11/07/go-vet-printf-family-check/).

## Patterns

### Test Tables

Các bài kiểm tra dựa trên bảng với [subtests](https://go.dev/blog/subtests) có thể là một mẫu thiết kế hữu ích cho việc viết các bài kiểm tra để tránh lặp lại mã khi logic kiểm tra cốt lõi là lặp đi lặp lại.

Nếu một hệ thống đang được kiểm tra cần được kiểm tra đối với _nhiều điều kiện_ trong đó một số phần của đầu vào và đầu ra thay đổi, thì nên sử dụng bài kiểm tra dựa trên bảng để giảm sự trùng lặp và cải thiện tính đọc.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestSplitHostPort(t *testing.T)

host, port, err := net.SplitHostPort("192.0.2.0:8000")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("192.0.2.0:http")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "http", port)

host, port, err = net.SplitHostPort(":8000")
require.NoError(t, err)
assert.Equal(t, "", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("1:8")
require.NoError(t, err)
assert.Equal(t, "1", host)
assert.Equal(t, "8", port)
```

</td><td>

```go
// func TestSplitHostPort(t *testing.T)

tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  {
    give:     "192.0.2.0:8000",
    wantHost: "192.0.2.0",
    wantPort: "8000",
  },
  {
    give:     "192.0.2.0:http",
    wantHost: "192.0.2.0",
    wantPort: "http",
  },
  {
    give:     ":8000",
    wantHost: "",
    wantPort: "8000",
  },
  {
    give:     "1:8",
    wantHost: "1",
    wantPort: "8",
  },
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    host, port, err := net.SplitHostPort(tt.give)
    require.NoError(t, err)
    assert.Equal(t, tt.wantHost, host)
    assert.Equal(t, tt.wantPort, port)
  })
}
```

</td></tr>
</tbody></table>

Bảng kiểm tra làm cho việc thêm ngữ cảnh vào các thông báo lỗi dễ dàng hơn, giảm logic trùng lặp và thêm các trường hợp kiểm tra mới.

Chúng tôi tuân theo quy ước rằng dãy cấu trúc được gọi là `tests` và mỗi trường hợp kiểm tra `tt`. Hơn nữa, chúng tôi khuyến khích rõ ràng hóa các giá trị đầu vào và đầu ra cho mỗi trường hợp kiểm tra với tiền tố `give` và `want`.

```go
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}

for _, tt := range tests {
  // ...
}
```

#### Avoid Unnecessary Complexity in Table Tests

Bảng kiểm tra có thể khó đọc và bảo trì nếu các bài kiểm tra phụ chứa các quả định điều kiện hoặc logic phân nhánh khác. Bảng kiểm tra **KHÔNG NÊN** được sử dụng bất cứ khi nào cần có logic phức tạp hoặc có điều kiện bên trong các bài kiểm tra phụ (tức là logic phức tạp bên trong vòng lặp `for`).

Bảng kiểm tra lớn, phức tạp gây hại cho tính đọc và tính bảo trì vì người đọc kiểm tra có thể gặp khó khăn trong việc gỡ lỗi các lỗi kiểm tra xảy ra.

Bảng kiểm tra như vậy nên được chia thành nhiều bảng kiểm tra hoặc nhiều hàm `Test...` riêng biệt.

Một số mục tiêu cần hướng đến là:

- Tập trung vào đơn vị hành vi hẹp nhất
- Tối thiểu hóa "sâu kiểm tra" và tránh những quả định có điều kiện (xem dưới đây)
- Đảm bảo rằng tất cả các trường bảng đều được sử dụng trong tất cả các kiểm tra
- Đảm bảo rằng tất cả logic kiểm tra chạy cho tất cả các trường hợp bảng

Trong ngữ cảnh này, "sâu kiểm tra" có nghĩa là "trong một bài kiểm tra nhất định, số lượng quả định liên tiếp cần phải được giữ lại" (tương tự như phức tạp vòng). Có "kiểm tra nông" có nghĩa là có ít mối quan hệ hơn giữa các quả định và, quan trọng hơn, các quả định đó ít có khả năng có điều kiện mặc định.

Cụ thể, bảng kiểm tra có thể trở nên rối rắm và khó đọc nếu chúng sử dụng nhiều đường dẫn phân nhánh (ví dụ: `shouldError`, `expectCall`, vv.), Sử dụng nhiều câu lệnh `if` cho các quả định giả định cụ thể (ví dụ: `shouldCallFoo`), hoặc đặt các hàm trong bảng (ví dụ: `setupMocks func(*FooMock)`).

Tuy nhiên, khi kiểm tra hành vi chỉ thay đổi dựa trên đầu vào thay đổi, có thể thích hợp để nhóm các trường hợp tương tự lại với nhau trong một bảng kiểm tra để minh họa tốt hơn cách hành vi thay đổi trên tất cả các đầu vào, thay vì phân chia các đơn vị có thể so sánh khác nhau thành các kiểm tra riêng biệt và làm cho chúng khó so sánh và đối chiếu hơn.

Nếu thân kiểm tra ngắn và rõ ràng, thì việc có một đường dẫn phân nhánh duy nhất cho các trường hợp thành công so với thất bại với một trường bảng như `shouldErr` để chỉ định kỳ vọng lỗi là chấp nhận được.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
func TestComplicatedTable(t *testing.T) {
  tests := []struct {
    give          string
    want          string
    wantErr       error
    shouldCallX   bool
    shouldCallY   bool
    giveXResponse string
    giveXErr      error
    giveYResponse string
    giveYErr      error
  }{
    // ...
  }

  for _, tt := range tests {
    t.Run(tt.give, func(t *testing.T) {
      // setup mocks
      ctrl := gomock.NewController(t)
      xMock := xmock.NewMockX(ctrl)
      if tt.shouldCallX {
        xMock.EXPECT().Call().Return(
          tt.giveXResponse, tt.giveXErr,
        )
      }
      yMock := ymock.NewMockY(ctrl)
      if tt.shouldCallY {
        yMock.EXPECT().Call().Return(
          tt.giveYResponse, tt.giveYErr,
        )
      }

      got, err := DoComplexThing(tt.give, xMock, yMock)

      // verify results
      if tt.wantErr != nil {
        require.EqualError(t, err, tt.wantErr)
        return
      }
      require.NoError(t, err)
      assert.Equal(t, want, got)
    })
  }
}
```

</td><td>

```go
func TestShouldCallX(t *testing.T) {
  // setup mocks
  ctrl := gomock.NewController(t)
  xMock := xmock.NewMockX(ctrl)
  xMock.EXPECT().Call().Return("XResponse", nil)

  yMock := ymock.NewMockY(ctrl)

  got, err := DoComplexThing("inputX", xMock, yMock)

  require.NoError(t, err)
  assert.Equal(t, "want", got)
}

func TestShouldCallYAndFail(t *testing.T) {
  // setup mocks
  ctrl := gomock.NewController(t)
  xMock := xmock.NewMockX(ctrl)

  yMock := ymock.NewMockY(ctrl)
  yMock.EXPECT().Call().Return("YResponse", nil)

  _, err := DoComplexThing("inputY", xMock, yMock)
  assert.EqualError(t, err, "Y failed")
}
```

</td></tr>
</tbody></table>

Sự phức tạp này làm cho việc thay đổi, hiểu và chứng minh tính đúng đắn của bài kiểm tra trở nên khó khăn hơn.

Mặc dù không có hướng dẫn cụ thể, tính đọc và bảo trì luôn được xem xét hàng đầu khi quyết định giữa Bảng kiểm tra so với các bài kiểm tra riêng biệt cho nhiều đầu vào/đầu ra của một hệ thống.

#### Parallel Tests

Kiểm tra song song, giống như một số vòng lặp chuyên biệt (ví dụ, những vòng lặp tạo ra goroutine hoặc chụp tham chiếu như một phần của thân vòng lặp),
phải chú ý gán biến vòng lặp một cách rõ ràng trong phạm vi của vòng lặp để đảm bảo rằng chúng giữ các giá trị mong muốn.

```go
tests := []struct{
  give string
  // ...
}{
  // ...
}

for _, tt := range tests {
  tt := tt // for t.Parallel
  t.Run(tt.give, func(t *testing.T) {
    t.Parallel()
    // ...
  })
}
```

Trong ví dụ trên, chúng ta phải khai báo một biến `tt` có phạm vi cho mỗi lần lặp vì việc sử dụng `t.Parallel()` bên dưới.
Nếu chúng ta không làm như vậy, hầu hết hoặc tất cả các bài kiểm tra sẽ nhận một giá trị không mong muốn cho `tt`, hoặc một giá trị thay đổi trong khi chúng đang chạy.

<!-- TODO: Explain how to use _test packages. -->

### Functional Options

Tùy chọn chức năng là một mẫu trong đó bạn khai báo một kiểu `Option` mờ mịt
để ghi lại thông tin trong một cấu trúc nội bộ nào đó. Bạn chấp nhận một số biến đa dạng
của các tùy chọn này và thực hiện hành động dựa trên thông tin đầy đủ được ghi lại bởi các tùy chọn trên
cấu trúc nội bộ.

Sử dụng mẫu này cho các đối số tùy chọn trong các hàm tạo và các API công cộng khác
mà bạn dự đoán sẽ cần mở rộng, đặc biệt là nếu bạn đã có ba hoặc
nhiều hơn ba đối số trên các hàm đó.

<table>
<thead><tr><th>Không nên</th><th>Nên</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Open(
  addr string,
  cache bool,
  logger *zap.Logger
) (*Connection, error) {
  // ...
}
```

</td><td>

```go
// package db

type Option interface {
  // ...
}

func WithCache(c bool) Option {
  // ...
}

func WithLogger(log *zap.Logger) Option {
  // ...
}

// Open creates a connection.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  // ...
}
```

</td></tr>
<tr><td>

Tham số cache và logger phải luôn được cung cấp, ngay cả khi người dùng muốn sử dụng mặc định.

```go
db.Open(addr, db.DefaultCache, zap.NewNop())
db.Open(addr, db.DefaultCache, log)
db.Open(addr, false /* cache */, zap.NewNop())
db.Open(addr, false /* cache */, log)
```

</td><td>

Options chỉ được cung cấp nếu cần.

```go
db.Open(addr)
db.Open(addr, db.WithLogger(log))
db.Open(addr, db.WithCache(false))
db.Open(
  addr,
  db.WithCache(false),
  db.WithLogger(log),
)
```

</td></tr>
</tbody></table>

Cách chúng tôi đề xuất để triển khai mẫu thiết kế này là sử dụng một giao diện `Option` chứa một phương thức không công khai, ghi lại các tùy chọn trên một cấu trúc `options` không công khai.

```go
type options struct {
  cache  bool
  logger *zap.Logger
}

type Option interface {
  apply(*options)
}

type cacheOption bool

func (c cacheOption) apply(opts *options) {
  opts.cache = bool(c)
}

func WithCache(c bool) Option {
  return cacheOption(c)
}

type loggerOption struct {
  Log *zap.Logger
}

func (l loggerOption) apply(opts *options) {
  opts.logger = l.Log
}

func WithLogger(log *zap.Logger) Option {
  return loggerOption{Log: log}
}

// Open creates a connection.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    cache:  defaultCache,
    logger: zap.NewNop(),
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}
```

Lưu ý rằng có một cách triển khai mẫu thiết kế này bằng cách sử dụng closures nhưng chúng tôi tin rằng mẫu trên cung cấp linh hoạt hơn cho tác giả và dễ dàng hơn để gỡ lỗi và kiểm thử cho người dùng. Cụ thể, nó cho phép so sánh các tùy chọn với nhau trong các bài kiểm thử và mocks, so với closures nơi điều này là không thể. Hơn nữa, nó cho phép các tùy chọn triển khai các giao diện khác, bao gồm `fmt.Stringer` cho phép các biểu diễn chuỗi có thể đọc được của các tùy chọn.

Xem thêm,

- [Self-referential functions and the design of options](https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html)
- [Functional options for friendly APIs](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis)

<!-- TODO: replace this with parameter structs and functional options, when to
use one vs other -->

## Linting

Quan trọng hơn bất kỳ tập hợp "phù hợp" nào của các công cụ kiểm tra cú pháp, hãy kiểm tra cú pháp một cách nhất quán trên toàn bộ mã nguồn.

Chúng tôi khuyên bạn nên sử dụng ít nhất các công cụ kiểm tra cú pháp sau, vì chúng giúp phát hiện các vấn đề phổ biến nhất và thiết lập một tiêu chuẩn cao cho chất lượng mã nguồn mà không cần thiết phải chi tiết:

- [errcheck](https://github.com/kisielk/errcheck) để đảm bảo rằng các lỗi được xử lý
- [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports) để định dạng mã và quản lý các import
- [golint](https://github.com/golang/lint) để chỉ ra các lỗi phong cách phổ biến
- [govet](https://pkg.go.dev/cmd/vet) để phân tích mã nguồn và kiểm tra các lỗi phổ biến
- [staticcheck](https://staticcheck.dev) để thực hiện các kiểm tra phân tích tĩnh khác nhau

### Lint Runners

Chúng tôi khuyên bạn nên sử dụng [golangci-lint](https://github.com/golangci/golangci-lint) là công cụ chạy kiểm tra cú pháp cho mã Go, chủ yếu là do hiệu suất của nó trong các dự án mã nguồn lớn và khả năng cấu hình và sử dụng nhiều công cụ kiểm tra cú pháp một cách hiệu quả. Repo này có một tập tin cấu hình [.golangci.yml](https://github.com/uber-go/guide/blob/master/.golangci.yml) mẫu với các công cụ kiểm tra cú pháp và cài đặt được khuyến nghị.

golangci-lint có [various linters](https://golangci-lint.run/usage/linters/) khác nhau có sẵn để sử dụng. Các công cụ kiểm tra cú pháp đã nêu ở trên được đề xuất làm tập hợp cơ bản, và chúng tôi khuyến khích các nhóm phát triển thêm bất kỳ công cụ kiểm tra cú pháp bổ sung nào mà họ cảm thấy hợp lý cho dự án của mình.
