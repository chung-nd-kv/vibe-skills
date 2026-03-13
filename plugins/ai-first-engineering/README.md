# AI-First Engineering Plugin

Plugin hỗ trợ quy trình phát triển AI-first với BDD (living documentation).

## Skills

| Skill | Role | Mô tả |
|-------|------|-------|
| **lean-prd** | Product | Viết PRD rút gọn tập trung Business Rules và Workflow |
| **prd-to-gherkin** | Engineer | Convert Lean PRD thành Gherkin feature files |
| **gherkin-to-code** | Engineer | Technical analysis + step definitions + implementation code từ feature files |
| **prd-to-mockup** | Shared | Tạo wireframe HTML từ Screen States trong PRD |

## Quy trình

```
Product: lean-prd → Lean PRD (Confluence)
                         ↓
Engineer: prd-to-gherkin → Feature files (Living docs repo)
                         ↓
Engineer: gherkin-to-code → Technical Solution → Step Definitions → Code
                         ↓
Product/Engineer: prd-to-mockup → Wireframe (validation)
```

## Cài đặt

Xem [GUIDE.md](GUIDE.md) cho hướng dẫn chi tiết.
