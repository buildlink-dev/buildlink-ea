# 央企专区 · 材料表分类映射 pipeline 产物

Plan 2(材料表 Excel 导入)的**分类映射决策**已完成,产物从易失的 session scratchpad
抢救到此。下一步 = **建新叶 + 落库导入**,直接读 `prepared/`。

## 上游输入(不在本目录,在 repo `data/` 根下)

- `data/基础校准拓展：材料名称中英文、规格、单位.xlsx` — 源材料表(17 大类、3724 行含规格变体,
  去重后 SPU 中文名 = 1646)。
- `data/央企专区_材料分类_待审.xlsx` — 可审交付物(映射表 / 待建 leaf / 统计;
  归属色标:继承绿 / 分类浅绿 / 救回蓝 / NEW占位橙)。

## prepared/ — 落库/建叶的直接输入(小、durable)

- `master_final.json` — **终态映射,1646 条,0 校验错,全部有 `final_code`**。
  每条:`{大类, zh, final_code, kind, conf, img, new_zh}`。
  `kind` 分布:继承 157 + 分类 1090 + 救回 70 + NEW占位 329 = 1646。
  - 继承/分类/救回(1317 条,80%)= 挂到真实 OVH leaf code。
  - NEW占位(329 条)= 先用 `parent_hint` 现有父 code 占位(可导入),`new_zh` 是待建叶名。
- `new_leaves.json` — **392 条待建叶**。每条:`{parent_hint_code, new_zh, new_en, count, examples}`。
  (与 master_final 的 329 NEW占位不同粒度:new_leaves 是按"待建叶"去重聚合。)

## raw/ — 源 dump(重、gitignored,可复算不入库)

OVH 生产库导出(方法见 memory ovh-production-host):
- `ovh_categories.tsv` — 全分类树(code/name/parent/level/is_leaf,6391 条)。**建新叶时查父 code 用这个**。
- `ovh_leaves.tsv` — 叶子层(4909 leaf)。
- `ovh_products.tsv` — 商品(10250,用于图片模糊匹配,导入阶段可不用)。

## working/ — pipeline 脚本 + 中间产物(重、gitignored,追溯/复算用)

`.py`/`.sql` 脚本 + 中间 JSON(`master_map.json` / `match_result.json` /
`trusted_inherit.json` / `inherit_157.json` / `gaps.txt`)+ `cls/` `cls2/` `salvage/`
(各批次子代理分类/救回原始输出)。

## 下一步 TODO(见 memory central-soe-zone-plan2-import)

1. 建 392 新叶(父 code 查 `raw/ovh_categories.tsv`)→ 冻结 → UPDATE master_final 里 329 占位重指真 leaf。
2. (可选)过滤噪声(暖通里的 计算器/窗帘/冰箱、工业气体 feedstock)、修 zh/en 错位。
3. 写幂等导入脚本:17 `zone_category` + SPU/SKU/attr/`zone_product`,
   入参用 code,幂等键 `(zone_code, zone_category_code, spu_code)`,`source=IMPORT`。
4. 干跑 → 导入 → 前端专区交易接线。
