# khoai-lang
"""Đánh giá kết quả và đưa ra khuyến nghị"""
    if df is None:
        return

    print_header("📊 ĐÁNH GIÁ KẾT QUẢ VÀ NHẬN XÉT")

    # 1. Tương quan giữa các biến
    print("\n1️⃣  PHÂN TÍCH TƯƠNG QUAN:")
    print("-" * 80)
    numeric_cols = df.select_dtypes(include=[np.number]).columns
    corr_matrix = df[numeric_cols].corr()

    print("\nMa trận tương quan giữa các biến:")
    print(corr_matrix.to_string())

    # Tìm các mối tương quan mạnh
    print("\n✓ Tương quan mạnh (|r| > 0.7):")
    strong_corr = False
    for i in range(len(corr_matrix.columns)):
        for j in range(i+1, len(corr_matrix.columns)):
            r = corr_matrix.iloc[i, j]
            if abs(r) > 0.7:
                print(f"   • {corr_matrix.columns[i]} ↔ {corr_matrix.columns[j]}: {r:.3f}")
                strong_corr = True
    if not strong_corr:
        print("   Không có tương quan mạnh (tất cả |r| ≤ 0.7)")

    # 2. Phân tích ROI (Lợi nhuận / Chi phí Marketing)
    print("\n2️⃣  PHÂN TÍCH HIỆU QUẢ MARKETING (ROI):")
    print("-" * 80)
    df['ROI'] = df['Lợi nhuận (Tỷ VNĐ)'] / df['Chi phí Marketing (Tỷ VNĐ)']
    print(f"   - ROI Trung bình: {df['ROI'].mean():.2f} (Lợi nhuận / Chi phí Marketing)")
    print(f"   - ROI Cao nhất: {df['ROI'].max():.2f} (Công ty {df.loc[df['ROI'].idxmax(), 'Công ty']:.0f})")
    print(f"   - ROI Thấp nhất: {df['ROI'].min():.2f} (Công ty {df.loc[df['ROI'].idxmin(), 'Công ty']:.0f})")
    print(f"   - Độ lệch chuẩn ROI: {df['ROI'].std():.2f}")

    # 3. Phân tích Profit Margin
    print("\n3️⃣  PHÂN TÍCH LỢI NHUẬN TRÊN DOANH THU (PROFIT MARGIN):")
    print("-" * 80)
    df['Profit_Margin'] = (df['Lợi nhuận (Tỷ VNĐ)'] / df['Doanh thu (Tỷ VNĐ)']) * 100
    print(f"   - Profit Margin Trung bình: {df['Profit_Margin'].mean():.2f}%")
    print(f"   - Profit Margin Cao nhất: {df['Profit_Margin'].max():.2f}% (Công ty {df.loc[df['Profit_Margin'].idxmax(), 'Công ty']:.0f})")
    print(f"   - Profit Margin Thấp nhất: {df['Profit_Margin'].min():.2f}% (Công ty {df.loc[df['Profit_Margin'].idxmin(), 'Công ty']:.0f})")

    # 4. Phân tích hiệu suất theo nhóm
    print("\n4️⃣  PHÂN TÍCH HIỆU SUẤT THEO NHÓM:")
    print("-" * 80)
    for label in df['Đánh giá hiệu suất (Label)'].unique():
        group_df = df[df['Đánh giá hiệu suất (Label)'] == label]
        count = len(group_df)
        avg_revenue = group_df['Doanh thu (Tỷ VNĐ)'].mean()
        avg_profit = group_df['Lợi nhuận (Tỷ VNĐ)'].mean()
        avg_customers = group_df['Số lượng khách hàng'].mean()
        print(f"\n   {label}:")
        print(f"      - Số công ty: {count}")
        print(f"      - Doanh thu TB: {avg_revenue:.2f} Tỷ VNĐ")
        print(f"      - Lợi nhuận TB: {avg_profit:.2f} Tỷ VNĐ")
        print(f"      - Khách hàng TB: {avg_customers:.0f}")

    # 5. Phân tích mối quan hệ Doanh thu - Khách hàng
    print("\n5️⃣  PHÂN TÍCH MỐI QUAN HỆ DOANH THU - KHÁCH HÀNG:")
    print("-" * 80)
    doanh_thu_col = 'Doanh thu (Tỷ VNĐ)'
    kh_col = 'Số lượng khách hàng'
    r_pearson, p_value = stats.pearsonr(df[doanh_thu_col], df[kh_col])
    print(f"   - Tương quan Pearson: {r_pearson:.3f}")
    print(f"   - P-value: {p_value:.6f}")
    if p_value < 0.05:
        print(f"   ✓ Mối quan hệ có ý nghĩa thống kê (p < 0.05)")
    else:
        print(f"   ⚠ Mối quan hệ không có ý nghĩa thống kê (p ≥ 0.05)")

    # 6. Xác định công ty tốt nhất và tệ nhất
    print("\n6️⃣  CÔNG TY NỔIBẬT:")
    print("-" * 80)
    best_company = df.loc[df['Lợi nhuận (Tỷ VNĐ)'].idxmax()]
    worst_company = df.loc[df['Lợi nhuận (Tỷ VNĐ)'].idxmin()]

    print(f"\n   🏆 CÔNG TY TỐT NHẤT (Lợi nhuận cao nhất):")
    print(f"      - Công ty: {best_company['Công ty']:.0f}")
    print(f"      - Doanh thu: {best_company['Doanh thu (Tỷ VNĐ)']:.0f} Tỷ VNĐ")
    print(f"      - Lợi nhuận: {best_company['Lợi nhuận (Tỷ VNĐ)']:.1f} Tỷ VNĐ")
    print(f"      - ROI: {df.loc[df['Công ty'] == best_company['Công ty'], 'ROI'].values[0]:.2f}")
    print(f"      - Đánh giá: {best_company['Đánh giá hiệu suất (Label)']}")

    print(f"\n   ⚠️  CÔNG TY CẦN CẢI THIỆN (Lợi nhuận thấp nhất):")
    print(f"      - Công ty: {worst_company['Công ty']:.0f}")
    print(f"      - Doanh thu: {worst_company['Doanh thu (Tỷ VNĐ)']:.0f} Tỷ VNĐ")
    print(f"      - Lợi nhuận: {worst_company['Lợi nhuận (Tỷ VNĐ)']:.1f} Tỷ VNĐ")
    print(f"      - ROI: {df.loc[df['Công ty'] == worst_company['Công ty'], 'ROI'].values[0]:.2f}")
    print(f"      - Đánh giá: {worst_company['Đánh giá hiệu suất (Label)']}")

    # 7. Nhóm công ty theo hiệu suất
    print("\n7️⃣  PHÂN LOẠI CÔNG TY:")
    print("-" * 80)
    for label in sorted(df['Đánh giá hiệu suất (Label)'].unique()):
        companies = df[df['Đánh giá hiệu suất (Label)'] == label]['Công ty'].values
        print(f"\n   {label}: Công ty {', '.join([str(int(c)) for c in companies])}")

    # 8. Khuyến nghị
    print("\n8️⃣  KHUYẾN NGHỊ & KẾT LUẬN:")
    print("-" * 80)

    # Tính toán các chỉ số
    avg_roi = df['ROI'].mean()
    high_roi_companies = df[df['ROI'] > avg_roi]
    low_roi_companies = df[df['ROI'] <= avg_roi]

    print(f"\n   ✓ Các công ty có ROI cao (> {avg_roi:.2f}):")
    for _, row in high_roi_companies.iterrows():
        print(f"      • Công ty {row['Công ty']:.0f}: ROI = {row['ROI']:.2f}")

    print(f"\n   ⚠️  Các công ty có ROI thấp (≤ {avg_roi:.2f}):")
    for _, row in low_roi_companies.iterrows():
        print(f"      • Công ty {row['Công ty']:.0f}: ROI = {row['ROI']:.2f} - Cần tối ưu chi phí marketing")

    print("\n   📌 Các gợi ý:")
    print(f"      • Tập trung vào chiến lược của các công ty có hiệu suất 'Tốt'")
    print(f"      • Xem xét tăng đầu tư marketing cho các công ty 'Bình thường' (ROI trung bình)")
    print(f"      • Rà soát chi phí marketing cho các công ty có ROI thấp")
    print(f"      • Mở rộng số lượng khách hàng - có tương quan dương với doanh thu")

    print("\n" + "="*80)

def main():
    """Hàm chính"""
    print("🚀 KHỞI ĐỘNG PHÂN TÍCH DỮ LIỆU TRONG MÔI TRƯỜNG KHDL_AI")
    print("="*80)
    print(f"Python version: {sys.version}")
    print(f"Working directory: {os.getcwd()}")
    print(f"Environment: KHDL_AI")
    print("="*80)

    # Chạy phân tích
    df = analyze_data()
    evaluate_results(df)

    print("\n✅ HOÀN THÀNH PHÂN TÍCH DỮ LIỆU!")
    print("📁 Các file đã được xử lý trong môi trường ảo KHDL_AI")
    print("🎯 Kết quả phân tích đã được hiển thị ở trên")

if __name__ == "__main__":
    main()
