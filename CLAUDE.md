# Claude Code — UXUI Agent Library

> อ่านไฟล์นี้ก่อนเริ่มทำงานเสมอ
> Owner: design@7solutions.co.th | Repo: github.com/sittipons-ike/uxui-agent-library

---

## 🎨 Design Skills Registry

Skills เก็บไว้ 2 ที่:

1. **`~/.claude/skills/`** — ติดตั้งแล้วในเครื่อง ใช้เป็น primary source
2. **`skills/`** ใน repo นี้ — source of truth ของทีม อัปเดตผ่าน `git pull && bash setup.sh`

### 📘 Team Skills (Thai, Figma-focused)

| Task | Skill Name | File |
|---|---|---|
| Audit/QA Figma (DS compliance) | `audit-ui` | `skills/audit-ui.md` |
| Plan UX / User flow / IA | `ux-skill` | `skills/ux-skill.md` |
| Implement UI from Blueprint | `ui-skill` | `skills/ui-skill.md` |
| Write microcopy | `ux-writing` | `skills/ux-writing.md` |

### ✨ Impeccable Skills (third-party — English, code-focused)

21 skills จาก [impeccable.style](https://impeccable.style) (Apache 2.0) — อยู่ที่ `~/.claude/skills/`

| Category | Skills |
|---|---|
| 🛠️ Foundation | `teach-impeccable`, `frontend-design` |
| 🔍 Quality & Review | `audit`, `critique`, `polish`, `harden`, `normalize`, `extract` |
| 🎨 Visual Tuning | `bolder`, `quieter`, `colorize`, `arrange`, `typeset`, `distill` |
| ✨ Motion & Delight | `animate`, `delight`, `overdrive`, `optimize` |
| 📱 Adapt & Improve | `adapt`, `onboard`, `clarify` |

---

## 🔌 Connected MCPs

- **figma-console** (`figma-console-mcp`) — อ่าน/เขียน/ตรวจ Figma โดยตรง

---

## 🎯 Skill Invocation Rules

### Trigger — ใช้ skill ไหนเมื่อไหร่

| User พูดว่า... | Skill ที่ใช้ |
|---|---|
| "audit", "review", "QA" + **Figma link** | `audit-ui` (team skill) |
| "audit", "review", "QA" + **code/web/built UI** | `/audit` (Impeccable) |
| "WCAG", "a11y", "contrast", "performance", "dark mode" | `/audit` (Impeccable) |
| "DS compliance", "token usage", "ใช้ DS ถูกมั้ย" | `audit-ui` (team skill) |
| "plan", "blueprint", "user flow", "IA" | `ux-skill` |
| "build UI", "implement", "spec", "tokens" + Figma link | `ui-skill` |
| "write copy", "rewrite", "microcopy", "CTA" | `ux-writing` |

### เลือกระหว่าง audit-ui vs /audit

```
ตรวจ UI?
  ├─ อยู่ใน Figma (design stage) → audit-ui
  │    - DS compliance เท่านั้น
  │    - comment เป็นภาษาไทยบน Figma
  │    - ใช้ก่อน handoff
  │
  └─ อยู่ใน code/browser (built) → /audit (Impeccable)
       - ครอบคลุม WCAG + perf + responsive
       - report ภาษาอังกฤษ
       - ใช้ก่อน ship
```

### วิธีใช้ skills

1. **อ่าน local file ก่อน** จาก `~/.claude/skills/{skill-name}.md`
2. **ทำตาม Execution Steps** ที่ระบุใน skill อย่างเคร่งครัด
3. **ใช้ Output Format** ที่ระบุ — ไม่สร้าง format เอง
4. **ไม่ละเมิด Constraints** ที่ระบุ

### Workflow Chaining

Skills ออกแบบให้ทำงานต่อกัน:

```
ux-strategist → [ui-implementation-specialist + ux-writer] → design-qa-auditor
```

ถ้า user ขอ "full design workflow" → ทำตามลำดับนี้

---

## 📋 Common Prompts

### ตรวจ Figma ก่อน handoff
```
Audit นี้: [Figma link]
ตรวจ DS compliance และ pin comment Critical issues บน Figma
```
→ ใช้ `audit-ui`

### วาง UX Blueprint
```
ช่วยวาง UX Blueprint สำหรับฟีเจอร์ [ชื่อ]
User goal: [...]
Business goal: [...]
```
→ ใช้ `ux-skill`

### Implement UI
```
Implement UI ตาม Blueprint นี้: [link]
Figma file: [link]
```
→ ใช้ `ui-skill`

### Rewrite copy
```
Rewrite copy ใน Figma นี้: [link]
Emotional state: [rushed/anxious/excited]
```
→ ใช้ `ux-writing`

---

## 🛡️ Safety Rules

- **ไม่แก้ไข skill files เอง** โดยไม่มี user confirm
- **ไม่ลบ Figma comments** ที่มีอยู่เดิม — ใช้ `get_comments` เช็คก่อน `post_comment`
- **ไม่ auto-approve** design QA ถ้ามี Critical issues
- ถ้าไม่แน่ใจ **ถาม user ก่อน** ดีกว่าเดาเอง

---

## 🔄 อัปเดต Skills

เมื่อทีม lead เพิ่ม skill ใหม่ใน repo:

```bash
git pull
bash setup.sh
```

---

## 📝 Team Info

- **Team:** UXUI Team — 7 Solutions
- **Repo:** github.com/sittipons-ike/uxui-agent-library
- **Last Updated:** 2026-05-10
