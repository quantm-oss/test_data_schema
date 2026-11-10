# Ngữ cảnh Tổ chức (Organization Context)

## 1. Hồ sơ Doanh nghiệp
- **Mã tổ chức (orgId)**: `bdf66a48c3ebfaec10d2feb3492099ea`
- **Lĩnh vực hoạt động**: Bán lẻ đa kênh (Omnichannel Retail) — kinh doanh thiết bị công nghệ, điện thoại, sản phẩm thương hiệu **Apple** và phụ kiện qua hệ thống cửa hàng vật lý (`offline`) và kênh trực tuyến (`online`, E-commerce, Social CRM, Zalo OA, Mobile App).
- **Thị trường & Địa lý (`countryCode: VN`)**:
  - Tỉnh/Thành phố chính (`city`): **Hà Nội, TP. Hồ Chí Minh, Đà Nẵng, Hải Phòng, Cần Thơ**.
  - Hỗ trợ lọc theo cấp Quận/Huyện (`district`) và Phường/Xã (`ward`).
- **Đơn vị tiền tệ**: VND (các mốc giá trị đơn hàng/chi tiêu phổ biến: 3 triệu, 10 triệu, 20 triệu, 30 triệu, 50 triệu VND).

## 2. Kênh Tương tác & Thanh toán
- **Kênh định danh & liên hệ khách hàng**: Số điện thoại (`mainPhone`), Email (`mainEmail`), Zalo (`mainZaloUserId`, Zalo OA `mainZaloOAId`), Facebook (`mainFacebookPsid`).
- **Kênh tương tác ưu tiên (`preferredChannel`) & Nguồn dữ liệu (`sources`, `lastUpdatedSource`)**: `Customer`, `PurchaseHistory`, `ZaloProfile`, `CustomerMembership`, `SocialCRM`, `CRM`, `Import`.
- **Phương thức thanh toán phổ biến**: Ví MoMo (`momo`), tiền mặt (`cash`), chuyển khoản (`transfer`), thẻ (`card`).

## 3. Chương trình Thành viên, Phân khúc & Chăm sóc Khách hàng
- **Hạng thành viên (`attributes.MembershipLevel`)**: `Silver`, `Gold`, `Diamond` (kèm thời điểm cấp hạng `attributeLongs.MembershipIssuedAt`).
- **Điểm thưởng Loyalty**: Điểm tích lũy hiện có (`attributeInts.LoyaltyPoint`) và điểm kiếm được (`attributeInts.EarnPoint`).
- **Mô hình eRFM (`enrichmentModels` với `model = "erfm"`)**:
  - Chỉ số hành vi & mua sắm: Độ gần mua hàng (`recency` / `erfm_recency`), tần suất mua (`frequency` / `erfm_frequency`), tổng chi tiêu trong kỳ (`monetary` / `erfm_monetary`), giá trị đơn trung bình (`aov` / `erfm_aov`), độ gần & tần suất tương tác (`recency_event`, `frequency_event`).
  - Điểm chuẩn hóa: `R_score`, `F_score`, `M_score`, `re_score`, `fe_score`, `eRFM_score` (thang điểm phổ biến từ 1–5 hoặc điểm tổng hợp eRFM).
- **Tag & Nhóm quan tâm hiện có**:
  - Tag khách hàng (`coreTagIds`): `Retail_VIP`, `Retail_Loyalty`, `Retail_Risk`, `Retail_Value`, `Retail_HOT`, `Retail_WARM`, `Retail_COLD`, `loyalty`.
  - Phân khúc/Nhóm quan tâm sản phẩm (`userSegmentKeys`, `lastSegmentKeys.PRODUCT_INTEREST`): ví dụ nhóm khách hàng quan tâm sản phẩm **Apple**.

## 4. Các Nhóm Thuộc tính Khách hàng Hỗ trợ Phân khúc (Segment)
- **Nhân khẩu học**:
  - Giới tính (`gender`): Nam (`male`), Nữ (`female`), chưa xác định (`unknown`).
  - Nhóm tuổi (`ageGroup`): `18-20`, `18-24`, `20-30`, `25-34`, `35-44`.
  - Ngày sinh (`birthdayDate`), tháng sinh (`birthdayMonth`), năm sinh (`birthYear`).
- **Lịch sử & Hành vi mua sắm**:
  - **Tích lũy mọi thời điểm**: Tổng giá trị đã mua (`totalPurchasedAmount`), tổng số lần mua (`totalPurchasedCount`), giá trị đơn hàng trung bình (`avgPurchasedAmount`).
  - **Giao dịch gần nhất & đầu tiên**: Thời điểm mua gần nhất (`lastPurchasedAt`), số ngày kể từ lần mua gần nhất (`lastPurchasedDays`), giá trị đơn gần nhất (`lastPurchasedAmount`), sản phẩm mua gần nhất (`lastPurchasedProducts`); thời điểm/giá trị/sản phẩm lần mua đầu tiên (`firstPurchasedAt`, `firstPurchasedAmount`, `firstPurchasedProducts`); danh mục ưu tiên (`preferredCategory`).
- **Hoạt động & Tương tác**:
  - Thời điểm hoạt động gần nhất (`lastActivityAt`), hoạt động đầu tiên (`firstActivityAt`), thời điểm tạo hồ sơ (`createdAt`), thời điểm chạm gần nhất (`lastTouchedAt`).
  - Tương tác Zalo OA: Đã quan tâm OA (`attributes.followedOA`), thời điểm tương tác Zalo OA gần nhất (`attributeLongs.lastInteractOA`).

## 5. Lưu ý Nghiệp vụ khi Gợi ý Câu hỏi
- **Tách biệt chỉ số tích lũy và khoảng thời gian**: `totalPurchasedAmount` và `totalPurchasedCount` là giá trị cộng dồn toàn thời gian (lifetime). Khi kết hợp với thời gian (ví dụ: "mua hàng trong 30 ngày gần đây"), hãy diễn đạt tách biệt hai điều kiện (ví dụ: *"Khách hàng có tổng chi tiêu trên 30 triệu đồng và có giao dịch mua hàng trong 30 ngày gần đây"*) thay vì gộp thành *"tổng chi tiêu trên 30 triệu trong 30 ngày gần đây"*.
- **Đặt tiêu chí cụ thể**: Luôn dùng ngưỡng số rõ ràng (số tiền, số đơn, số ngày, điểm số, hạng thẻ `Gold`/`Diamond`/`Silver`) thay vì các cụm từ mơ hồ như "khách VIP", "mua nhiều", "khách thân thiết" không kèm điều kiện.
- **Không gợi ý dữ liệu ngoài hệ thống**: Không đề cập đến hành vi mở email chiến dịch, click quảng cáo hoặc giỏ hàng bỏ quên vì chưa có trong hồ sơ phân khúc.
