# UXUI Agent Library

> Skill library สำหรับทีม Designer ใช้ Claude Code ร่วมกับ Figma
> Owner: design@7solutions.co.th

---

## Project นี้คืออะไร

เป็น repository กลางของทีม UXUI สำหรับเก็บ:
- **Skills** — ไฟล์ที่สอน Claude วิธีทำงาน UX/UI โดยเฉพาะ
- **ONBOARDING.md** — คู่มือติดตั้งสำหรับทีม
- **setup.sh** — script ติดตั้ง skills ลงเครื่องอัตโนมัติ

---

## Skills ที่มีอยู่

| ไฟล์ | ชื่อ skill | หน้าที่ |
|---|---|---|
| `skills/audit-ui.md` | design-qa-auditor | ตรวจ Figma DS compliance ก่อน handoff |
| `skills/ux-skill.md` | ux-strategist | วาง User Flow + Information Architecture |
| `skills/ui-skill.md` | ui-implementation-specialist | Map component + token จาก Blueprint |
| `skills/ux-writing.md` | ux-writer | เขียน / rewrite microcopy บน UI |

---

## วิธีใช้ Skills

Skills ติดตั้งแล้วที่ `~/.claude/skills/` — ใช้ได้ทุก session โดยพิมพ์ตาม use case ปกติ
Claude จะเรียก skill ที่ถูกต้องอัตโนมัติตาม trigger ของแต่ละ skill

---

## วิธีเพิ่ม Skill ใหม่

1. สร้างไฟล์ `skills/ชื่อ-skill.md` ตาม format เดิม
2. `git add` → `git commit` → `git push`
3. ทีมรัน `git pull && bash setup.sh` เพื่ออัปเดต

---

## Connected Tools

- **Figma MCP** (`figma-console-mcp`) — ต่อ Figma โดยตรง อ่าน/เขียน/ตรวจ design ได้
- ตั้งค่าที่ `~/Library/Application Support/Claude/claude_desktop_config.json`

---

## Team

- **Lead:** sittipon.s@7solutions.co.th
- **Repo:** github.com/sittipons-ike/uxui-agent-library
- **Last updated:** 2026-05-10
