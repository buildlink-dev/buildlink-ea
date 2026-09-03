# PDF 商品图册整理与导入模板

适用来源：

- `厦鹰图册.pdf`
- `山河消防图册(2).pdf`

整理路线：不要直接从 PDF 入库。先由人工按 `_templates/*.csv` 整理成结构化数据，再由导入脚本校验并写入 SPU/SKU 表。

## 拆分口径

- SPU：一款产品或一个清晰产品型号族。厦鹰通常按产品卡拆；山河消防通常按页面商品块拆。
- SKU：可采购、报价、询价时需要区分的具体型号/规格。参数表每一行“型号/规格”通常就是一个 SKU。
- 属性：压力、通径、适用介质、温度、接口、容量、材质等放 `attrs.csv`。可选择规格轴填 `selectable=true`。
- 图片：已裁好的图填 `image_file`；未裁图先填 PDF 页码和 `crop_note`，后续补图。

## 建议批次目录

```text
data/manual_catalog_imports/2026-07-xiaying-shanhe/
  raw/
    厦鹰图册.pdf
    山河消防图册(2).pdf
  working/
    廖总整理说明.md
    分类映射待确认.csv
  images/
    XIAYING-H42H/main.jpg
    SHANHE-SQD/main.jpg
  prepared/
    manifest.csv
    spus.csv
    skus.csv
    attrs.csv
    images.csv
  reports/
    validation_errors.csv
    import_summary.md
```

脚本只读取 `prepared/`；`working/` 是人工整理过程文件。

## 模板文件

- `_templates/manifest.csv`：批次来源。
- `_templates/spus.csv`：一行一个 SPU。
- `_templates/skus.csv`：一行一个 SKU。
- `_templates/attrs.csv`：一行一个 SPU/SKU 属性。
- `_templates/images.csv`：一行一张图片或待裁图标记。

## 外部 key

每个 SPU/SKU 必须有稳定 key，方便修正后重复导入。

- 厦鹰 SPU：`XIAYING-P{页码}-B{块号}-{型号}`
- 山河 SPU：`SHANHE-P{页码}-B{块号}-{型号或短名}`
- SKU：`{external_spu_key}-R{参数表行号}`

示例：

- `XIAYING-P120-B01-H42H`
- `SHANHE-P25-B01-SQD`
- `SHANHE-P25-B01-SQD-R01`

## 后续脚本建议

建议新增 `backend/scripts/import_products_manual_catalog.py`：

```bash
cd backend
python scripts/import_products_manual_catalog.py \
  --batch ../data/manual_catalog_imports/2026-07-xiaying-shanhe \
  --dry-run

python scripts/import_products_manual_catalog.py \
  --batch ../data/manual_catalog_imports/2026-07-xiaying-shanhe \
  --commit
```

校验重点：

- `category_code` 必须存在。
- 每个 SPU 至少一个 SKU。
- 每个 SPU 只能一个默认 SKU。
- `external_spu_key/external_sku_key` 不可重复。
- CSV 列数和必填字段必须正确。

导入策略：

- `spus.csv` 写 `products`。
- `skus.csv` 写 `product_skus`。
- `attrs.csv` 写 `product_attrs`。
- `images.csv` 写 `product_images`。
- 默认首批导入为 `DRAFT`，人工确认后再上架。
