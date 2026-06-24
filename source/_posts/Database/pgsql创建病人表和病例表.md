---
title: pgsql创建病人表和病例表
tags: []
comments: true
toc: true
toc_number: true
toc_style_simple: false
copyright: true
copyright_author: iehtian
mathjax: false
katex: false
aplayer: false
highlight_shrink: false
aside: true
abcjs: false
noticeOutdate: false
date: 2025-12-12 20:03:17
updated: 2025-12-12 21:53:44
categories:
keywords:
description:
top_img:
cover: https://picsum.photos/id/111/800/450
copyright_author_href:
copyright_url:
copyright_info:
---
```
-- 创建病人表
CREATE TABLE patients (
    patient_id SERIAL PRIMARY KEY,           -- 病人唯一编号（自增主键）
    patient_name VARCHAR(100) NOT NULL,      -- 姓名
    birth_date DATE NOT NULL,                -- 出生日期
    gender CHAR(1) NOT NULL,                 -- 性别 ('M'=男, 'F'=女)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  -- 记录创建时间
    CONSTRAINT chk_gender CHECK (gender IN ('M', 'F'))  -- 性别约束
);

-- 创建病例表
CREATE TABLE medical_records (
    record_id SERIAL PRIMARY KEY,            -- 病例唯一编号（自增主键）
    patient_id INTEGER NOT NULL,             -- 病人编号（外键）
    visit_date DATE NOT NULL,                -- 就诊日期
    visit_time TIME NOT NULL,                -- 就诊时间
    age_at_visit INTEGER NOT NULL,           -- 就诊时年龄
    gender CHAR(1) NOT NULL,                 -- 性别
    remarks TEXT,                            -- 就诊信息（备注）
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  -- 记录创建时间
    CONSTRAINT fk_patient FOREIGN KEY (patient_id) 
        REFERENCES patients(patient_id) 
        ON DELETE CASCADE,                   -- 删除病人时级联删除病例
    CONSTRAINT chk_record_gender CHECK (gender IN ('M', 'F'))
);

-- 创建索引以提高查询效率
CREATE INDEX idx_patient_name ON patients(patient_name);
CREATE INDEX idx_patient_id ON medical_records(patient_id);
CREATE INDEX idx_visit_date ON medical_records(visit_date);

-- 添加注释
COMMENT ON TABLE patients IS '病人基本信息表';
COMMENT ON TABLE medical_records IS '病例记录表';
COMMENT ON COLUMN patients.patient_id IS '病人唯一编号';
COMMENT ON COLUMN medical_records.patient_id IS '关联病人表的外键';
```