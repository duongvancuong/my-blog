---
title: Multi-Tenant Key of SaaS.
date: "2021-08-14T23:46:37.121Z"
template: "post"
draft: false
slug: "/posts/multi-tenant-key-of-saas/"
category: "Database"
tags:
  - "Database"
description: "Khi thực hiện phát triển 1 dự án multi-tenant SaaS thì việc lựa chọn giải pháp xây dựng database phục vụ cho nhiều tenant để đảm bảo các tiêu chí về bảo mật, hiểu suất và khả năng bảo trì khá quan trọng, bài viết này sẽ giới thiệu một vài phương pháp để lựa chọn việc xây dựng hệ thống database phù hợp cho từng trường hợp cụ thể"
---

## Single Tenant:
- Single Tenant là 1 instance application và hỗ trợ infrastructure cho một khách hàng. Khách hàng có một phiên bản ứng dụng và database độc lập và không có 1 sự chia sẽ nào ở đây. 


## Multi-Tenant:
- Có nghĩa là 1 instance application và các cơ sở hạn tầng của nó phục vụ cho nhiều khách hàng. Mỗi khách hàng chia sẽ ứng dụng. Dữ liệu của mỗi tenant được tách biệt vào không thể nhìn thấy bởi tenant khác. Người dùng sẽ cảm thấy rằng họ là người duy nhất sử dụng ứng dụng. Mỗi người dùng sẽ có dữ liệu riêng của họ và chỉ họ mới có thể  nhìn thấy và sửa đổi nó.
- Với việc triển khai như vậy nhà cung cấp service (owner SaaS) sẽ tiết kiệm được chi phí triển khai ứng dụng và cơ sở hạ tầng cho nhiều tenant sử dụng.

## Các vấn đề  về  Database khi triển khai Multi-Tenant:

1. Security
2. Maintainability
3. Scalability

- Đối với Single-Tenant thì những vấn đề trên cũng quan trọng nhưng đối với multi-tenant thì nó có thêm những điều cần lưu ý, vì sẽ phải lưu trữ data cho nhiều tenant và số lượng mỗi tenant có thể tăng lên nhanh chóng.

## #1 Security:

- Bảo mật dữ liệu rất quan trọng trong tất cả các hệ thống, nhưng trong hệ thống nhiều tenant thì sẽ có thêm 1 khía cạnh khác đó là dữ liệu ở đây sẽ chứa của nhiều tenant thay vì chỉ 1 tenant như trong hệ thống dành cho 1 tenant riêng lẻ.
- Nếu cung cấp phần mềm cho nhiều tenant thì yêu cầu cơ bản là 1 tenant này sẽ không thể xem hoặc tương tác tới data của 1 tenant khác. Nếu có phá vỡ điều đó thì chắc chắn hệ thống này đã vi phạm tiêu chuẩn về bảo mật.


## #2 Maintainability:

- Đối với single-tenant system. Chúng ta có thể có plan để  thực hiện backup dữ liệu. Chúng ta có thể thực hiện các jobs hằng đêm để thực hiện các tác vụ maintenance như rebuid index, reorganization, kiểm tra tính toàn vẹn của dữ liệu, để giữ cho hệ thống hoạt động cách tốt nhất. Nhưng đối với hệ thống multi-tenant thì câu hỏi đặt ra ở đây là: "Công việc bảo trì sẽ như thế nào khi số lượng tenant tăng lên quá nhanh?" từ 1 tenant chúng ta có sẽ có đến hàng trăn hoặc hằng ngàn tenant trong tương lai. Vậy nên quá trình bảo trì sẽ trở nên khó khăn hơn và tốn thời gian hơn.

## #3 Scalability:

- Đối với single-tenant system, có thể chúng ta sẽ tự hỏi: "Khi khối lượng dữ liệu tăng lên, làm thế nào để duy trì 1 hiệu suất ổn định?"
- Đối với multi-tenant system thì câu hỏi ở đây sẽ là: "Khi add nhiều tenant và dữ liệu của mỗi tenant tăng lên thì làm thế nào để duy trì hiệu suất ổn định?"

Điểm khác nhau ở đây là 1 tenant với 10 triệu bản ghi và 1000 tenant với 10 triệu bản ghi cho mỗi tenant. Là người cung cấp service chúng ta sẽ luôn muốn có càng nhiều tenant thì càng tốt vào con số đó sẽ luôn là ẩn số. Và để  đảm bảo ứng dụng luôn hoạt động với hiệu suất tốt là 1 bài toán khó.
 

## Phương pháp tiếp cận và thiết kế  Database:

Theo tìm hiểu và nghiên cứu thì ở đây tôi sẽ chỉ ra 4 cách tiếp cận:
1. Single database, shared schema.
2. Single database, separate schema.
3. Database per tenant.
4. Multiple databases, multiple tenants per database, shared schema.

## Phương pháp #1: Single database, shared schema:
- Một database để lưu trữ cho toàn bộ tất cả tenant.
- Dữ liệu của mỗi tenant được lưu trữ trên cùng 1 table như nhau.
- Các table chứa dữ liệu dành cho từng tenant sẽ gồm 1 column để các xác định nó thuộc về tenant nào. (tenant_id)

Với cách này thì sẽ như thế nào với 3 vấn đề trên.

### Security:
- [❌ ] Rủi ro có thể xảy ra là để lộ dữ liệu or cập nhật sai dữ liệu cho 1 tenant khác. (nếu developer code thiếu điều kiện WHERE với tenant_id thì thôi xong toang cmnr)
- [NOTE] => Row-Level Security (RSL) có thể được sử dụng để kiểm soát việc truy cập vào từng row của 1 table. Chúng ta sẽ tạo một hàm để áp dụng filter dựa trên tenant_id và sau đó tạo 1 policy để áp dụng filter đó trên các table. Miễn là duy trì policy đó thì các truy vấn và cập nhật sẽ được áp dụng, developer sẽ không cần phải thêm filter vào từng câu truy vấn SQL nữa. 
- [❌ ] Không tách biệt data cho mỗi tenant.

### Maintainability:
- [✔️] Việc chỉ có 1 database schema to maintain sẽ đơn giản hơn và chỉ cần áp dụng 1 lần.
- [✔️] Việc quản lý High Availability/Disaster Recovery/ maintainance operation/monitoring sẽ chỉ dành cho 1 database.
- [✔️] Việc thực hiện coding sẽ ít phức tạp hơn khi chỉ có 1 schema, 1 database để connect.
- [✔️] Việc thêm 1 tenant mới vào database sẽ đơn giản hơn.
- [❌ ] Mọi câu query hoặc modification data đều phải định nghĩa một tenant_id
- [❌ ] Phải thực hiện RLS policy lên những table mới.
- [❌ ] Không thể đơn giản để có thể restore 1 single tenant.

### Scalability:
- [❌ ] Rủi ro về hiệu suất. Vì phải chia sẻ cùng tài nguyên với những tenant khác.
- [❌ ] Số lượng tenant tăng lên và lượng dữ liệu của mỗi tenant cũng tăng lên ảnh hưởng tới việc maintanance cũng trở nên khó khăn hơn.

## Phương pháp #2: Single Database, Separate Schema.
- Một database để lưu trữ cho toàn bộ tất cả tenant.
- Phân tách các table cho mỗi tenant, mỗi tenant cho một schema riêng.

Với cách này thì sẽ như thế nào với 3 vấn đề trên.

### Security:
- [✔️] Dữ liệu của mỗi tenant có thể tách biệt hơn (nhưng vẫn chung 1 database)
- [✔️] Không cần RLS. giảm thiểu rủi ro khi quên điều kiện WHERE với mỗi tenant cụ thể.
- [NOTE] => Các câu truy vấn cần dynamic để có thể matching đúng với table name hoặc connect sử dụng 1 tenant user account với default schema được set cho schema của tenant đó.
- [❌ ] Vẫn có rủi ro khi thực hiện query sai schema.

### Maintainability:
- [✔️] Việc quản lý High Availability/Disaster Recovery/ maintainance operation/motioring sẽ chỉ dành cho 1 database.
- [✔️] Chia được scope và control cụ thể cho từng tenant. giúp hoạt động maintainance dễ dàng hơn.
- [❌ ] Việc cập nhật schema diễn ra nhiều hơn thay vì chỉ 1 schema như single-tenant, phải thực hiện cho n tenant.
- [❌ ] Câu Query sẽ phức tạp hơn.
- [❌ ] Việc restore data cho 1 tenant khó khăn ( mặc dụ đã tốt hơn so với phương án 1)
- [❌ ] Khi thêm mới 1 tenant thì cần thêm 1 schema/user account cũng cần được tạo.
- [❌ ] Khi số lượng tenant lớn lên thì sẽ có 1 lượng lớn database object được tạo để quản lý vào maintain.

### Scalability:
- [✔️] Dữ liệu được chia trong các table nhỏ hơn, và index cũng sẽ nhỏ hơn.
- [✔️] Tối ưu hóa được ở cấp độ schema.
- [❌ ] Rủi ro về hiệu suất. vì phải chia sẽ cùng tài nguyên với những tenant khác.


## Phương pháp #3: Database Per Tenant:
- Mỗi tenant có 1 cơ sở dữ liệu riêng của họ.

### Security:
- [✔️] Cấp độ tách biệt dữ liệu cao nhất, hỗ trợ cho việc dù  là chia sẽ server ứng dụng hoặc tách biệt luôn.

### Maintainability:
- [✔️] Việc Maintenance có thể được quản lý là thực hiện cho mỗi tenant.
- [✔️] Có thể dễ dàng restore/relocate/clear data của mỗi tenant.
- [✔️] Câu Query không còn phức tạp nữa.
- [❌ ] Khi lượng tenant lớn lên thì sẽ có 1 lượng lớn database để quản lý vào maintain.
- [❌ ] Việc maintain có phức tạp hơn trong việc xác định ứng dụng kết nối đến database của tenant nào đó.

### Scalability:
- [✔️] Tối ưu hóa được ở cấp độ database.
- [✔️] Giải quyết được vấn đến hiệu suất khi tách biệt database cho mỗi tenant.

## Phương pháp #4: Multiple Database, Multiple Tenants Per Database, Shared Schema:
- Thằng con lai giữa phương pháp #1 và #3.
- Các tenant chia sẽ database và schema, nhưng cũng được tách biệt với multiple database nữa.

### Security:
- [✔️] Nhìn chung thì vẫn có thể tách biệt data của các tenant so với phương pháp #1.
- [❌ ] Một số tenant sẽ vẫn phải chia sẽ database và schema nhưng ở phương pháp #1.

### Maintainability:
- [✔️] Có thể di chuyển data của tenant và restore nó (nhưng phức tạp hơn phương pháp #3)
- [❌ ] Chi phí maintenance cao hơn phương pháp #1.

### Scalability:
- [✔️] Có thể Scale-out và Scale-up.
- [✔️] Có thể lựa chọn cân bằng giữa chi phí và hiệu suất (ví dụ như: nhiều tenant/ít server hoặc ít tenant/nhiều server)

## Đưa ra lựa chọn:

Dựa vào tình hình cụ thể cũng như của mỗi nhà đầu tự sẽ lựa chọn những phương pháp thích hợp.
ví dụ như:
- Nếu security và tách biệt data là ưu tiên hàng đầu cho dịch vụ của bạn thì phương pháp #3 sẽ tốt.
- Nếu bạn mong đợi lượng tenant tăng lên và cân bằng về chi phí cũng như sự mở rộng thì phương pháp #4 sẽ hợp lý.
- Nếu bạn đang xây dựng cho một lượng nhỏ tenant và khả năng tăng chậm về mặt dữ liệu thì phương pháp #1 hoặc #2 sẽ hợp lý.
- Nếu bạn mong muốn có thể mở rộng tối đa thì phương pháp #3 sẽ tốt.

Trên là những phương pháp để có thể xây dựng 1 hệ thông cơ sở dữ liệu cho loại hình Multi-Tenant chia khóa cho những ứng dựng SaaS.