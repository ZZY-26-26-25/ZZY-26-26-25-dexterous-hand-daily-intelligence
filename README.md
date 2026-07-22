# Awesome Dexterous Hand Intelligence

一个持续更新的具身智能与灵巧手研究知识库。

本项目采用 Awesome List + Structured Database 的形式组织灵巧手生态，参考 Awesome UMI 这类公开目录：通过结构化数据记录设备、论文、代码、数据集、产品和演示，并进一步生成网页检索界面。citeturn0search2

## 内容结构

```
README.md
├── Hardware Database
├── Teleoperation Database
├── UMI / Data Collection Database
├── Tactile Database
├── Dataset Database
├── Algorithm Database
├── Company & Product Tracking
└── Daily Intelligence Archive
```

## 分类体系

### 🤖 Dexterous Hand Hardware
- Shadow Hand
- Allegro Hand
- LEAP Hand
- Inspire Hand
- Wuji Hand
- Revo3
- MIDAS Hand
- OmniHand
- LinkerHand
- Open-source/custom hands

### 🖐 Data Collection & Teleoperation
- UMI
- UMI-like interfaces
- Wearable gloves
- Exoskeleton hands
- Whole-body teleoperation
- Human demonstration capture

### ✋ Tactile / Force
- Tactile sensors
- Vision tactile
- Force torque sensing
- Motor-current sensing
- Haptic feedback

### 🧠 Learning
- VLA
- Imitation Learning
- ACT
- Diffusion Policy
- Retargeting
- Tactile Policy
- World Models

## Database Fields

Each entry stores:

- Name
- Category
- Original URL
- Paper / Project / GitHub
- Hardware form
- DoF
- Sensors
- Finger tracking
- Wrist 6D tracking
- Tactile modality
- Force feedback
- Open-source status
- License
- Reproduction difficulty
- Procurement status
- Research value

## Update Workflow

Every intelligence update:

1. Add `reports/YYYY-MM-DD.md`
2. Update `data/index.json`
3. Update `data/items.csv`
4. Update comparison tables
5. Update procurement tracking
6. Update reproduction notes

## Roadmap

- [x] Daily intelligence archive
- [x] Structured taxonomy
- [x] Hardware comparison database
- [ ] GitHub Pages interactive catalog
- [ ] Search/filter frontend
- [ ] Community contribution workflow
