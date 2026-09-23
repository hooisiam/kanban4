# Color Palette Skill for Claude Code

A comprehensive skill for creating distinctive, accessible color palettes for UI/web design that avoid generic AI aesthetics.

## Overview

This skill helps Claude create color palettes that:
- ✅ Stand out from generic AI-generated designs
- ✅ Meet WCAG accessibility standards (AA/AAA)
- ✅ Are contextually appropriate for specific domains
- ✅ Avoid overused patterns (purple gradients, orange-teal, etc.)
- ✅ Follow professional design system principles

## What's Included

### 📄 SKILL.md
Comprehensive workflow guide with:
- 6-step color palette creation process
- 3 different approaches (domain-specific, brand-based, custom)
- Detailed examples for Tech/SaaS, E-commerce, and Dark Mode
- Anti-AI design checklist
- Common questions and advanced usage patterns

### 🛠️ Scripts

**`scripts/check_contrast.py`** - WCAG Contrast Validator
```bash
python scripts/check_contrast.py #000000 #FFFFFF normal
```
- Validates color combinations against WCAG AA/AAA standards
- Checks normal text, large text, and UI components
- Provides clear pass/fail feedback with recommendations

**`scripts/generate_palette.py`** - Color Scale Generator
```bash
python scripts/generate_palette.py #3B82F6 scale
```
- Generates Tailwind-style color scales (50-950 shades)
- Creates color harmonies (complementary, triadic, analogous)
- Uses HSL color space for perceptually uniform results

### 📚 References

**`references/color-palette-ui-design-reference.md`** - Comprehensive color guide (990+ lines) including:
- Color theory fundamentals (5 harmony types)
- Color psychology for 9 major colors
- **60+ curated real-world palettes** organized by domain:
  - Tech/SaaS (4 examples)
  - E-commerce (5 examples)
  - Healthcare (3 examples)
  - Finance (3 examples)
  - Creative/Portfolio (4 examples)
  - Food/Restaurant (4 examples)
- Design system approaches (Material Design, Tailwind, Ant Design)
- WCAG accessibility guidelines
- **Common AI Design Pitfalls** with strategies to avoid them
- 50+ cited sources from authoritative design resources

## Installation

### Method 1: From Release (Easiest) ⭐

Perfect for end users who just want to use the skill:

1. **Download the `.skill` file** from the [latest release](https://github.com/nhatmobile1/color-palette-skill/releases)

2. **Find your Claude Code skills directory:**
   ```bash
   ls ~/.claude/plugins/cache/anthropic-agent-skills/example-skills/
   ```
   Note the hash directory (e.g., `69c0b1a06741`)

3. **Extract the skill:**
   ```bash
   # Navigate to skills directory
   cd ~/.claude/plugins/cache/anthropic-agent-skills/example-skills/[YOUR_VERSION]/skills/

   # Extract the .skill file (it's a zip archive)
   unzip ~/Downloads/color-palette.skill
   ```

4. **Verify installation:**
   ```bash
   ls ~/.claude/plugins/cache/anthropic-agent-skills/example-skills/[YOUR_VERSION]/skills/color-palette
   ```

The skill will automatically be available in all future Claude Code sessions.

### Method 2: From Source (For Developers)

Perfect if you want to modify the skill or contribute:

1. **Clone the repository:**
   ```bash
   git clone https://github.com/nhatmobile1/color-palette-skill.git
   ```

2. **Find your Claude Code skills directory:**
   ```bash
   ls ~/.claude/plugins/cache/anthropic-agent-skills/example-skills/
   ```
   Note the hash directory (e.g., `69c0b1a06741`)

3. **Copy to Claude Code skills directory:**
   ```bash
   # Replace [YOUR_VERSION] with actual hash from step 2
   cp -r color-palette-skill ~/.claude/plugins/cache/anthropic-agent-skills/example-skills/[YOUR_VERSION]/skills/color-palette
   ```

4. **Verify installation:**
   ```bash
   ls ~/.claude/plugins/cache/anthropic-agent-skills/example-skills/[YOUR_VERSION]/skills/color-palette
   ```

### Method 3: Standalone Scripts

If you prefer to use the scripts independently without Claude Code:

```bash
git clone https://github.com/nhatmobile1/color-palette-skill.git
cd color-palette-skill/scripts
python check_contrast.py #000000 #FFFFFF normal
python generate_palette.py #3B82F6 scale
```

## Usage

### Automatic Triggering

The skill automatically triggers when you ask Claude:
- "Create a color palette for my e-commerce website"
- "I need colors for a healthcare app"
- "Design a color scheme for my SaaS dashboard"
- "Help me pick accessible colors for my project"
- "Generate a color palette that doesn't look AI-generated"

### Manual Invocation

```bash
/color-palette
```

### Example Workflows

**Example 1: E-commerce Site**
```
User: "I need warm, earthy colors for a sustainable fashion e-commerce site"

Claude will:
1. Reference the E-commerce → Fashion & Apparel palettes
2. Select earthy tones (teal, terracotta, gold)
3. Validate all contrast ratios
4. Provide complete color system with usage guidance
5. Explain how it avoids generic AI patterns
```

**Example 2: SaaS Dashboard**
```
User: "Create a professional color palette for a B2B SaaS analytics dashboard"

Claude will:
1. Read Tech/SaaS section from references
2. Choose Professional SaaS Dashboard palette
3. Extend with full color scales using generator
4. Validate WCAG compliance
5. Structure complete design system
```

## Key Features

### 🎯 Avoids AI Design Pitfalls
- Identifies overused patterns (purple/blue gradients, orange-teal)
- Provides domain-specific alternatives
- Anti-AI checklist with 8 validation questions

### ♿ Accessibility-First
- All palettes validated for WCAG AA/AAA compliance
- Contrast checker script included
- Guidance for text, UI components, and interactive elements

### 🎨 Professional Design System Structure
- Primary, secondary, accent, neutral, and semantic colors
- Tailwind-style color scales (50-950)
- Semantic naming conventions
- 60-30-10 rule guidance

### 🌍 Domain-Specific Expertise
Real-world curated palettes for:
- Tech/SaaS (trust, innovation, professionalism)
- E-commerce (conversion, urgency, approachability)
- Healthcare (calm, trust, cleanliness)
- Finance (stability, wealth, security)
- Creative/Portfolio (personality, boldness, uniqueness)
- Food/Restaurant (appetite stimulation, warmth)

## Requirements

### Python Dependencies

The scripts require Python 3.6+ with no external dependencies (uses only standard library).

For the palette generator, ensure you have:
```bash
python3 --version  # Should be 3.6 or higher
```

Both scripts use only Python standard library modules:
- `colorsys` - Color space conversions
- `sys` - Command-line argument handling
- `typing` - Type hints

## Examples

### Check Contrast
```bash
# Normal text on white background
python scripts/check_contrast.py #1A1A1A #FFFFFF normal

# Output:
# Contrast Ratio: 16.10:1
# WCAG Compliance:
#   Level AA: ✓ PASS
#   Level AAA: ✓ PASS
#   UI Components (3:1): ✓ PASS
```

### Generate Color Scale
```bash
# Create Tailwind-style scale
python scripts/generate_palette.py #3B82F6 scale

# Output:
# Color Scale for #3B82F6:
# ========================================
#    50: #fffffe
#   100: #ffffff
#   200: #ecf2fc
#   ...
#   950: #00307e
```

### Generate Color Harmonies
```bash
# Complementary colors
python scripts/generate_palette.py #3B82F6 complementary

# Triadic colors
python scripts/generate_palette.py #3B82F6 triadic

# Analogous colors
python scripts/generate_palette.py #3B82F6 analogous
```

## Contributing

Contributions are welcome! Areas for improvement:
- Additional domain-specific palettes
- More color harmony algorithms
- Cultural color considerations for different regions
- Advanced accessibility features (colorblind simulation)
- Integration with design tools (Figma, Sketch plugins)

## License

MIT License - See LICENSE file for details

## Credits

### Research & References
This skill is built on research from 50+ authoritative design sources including:
- Material Design 3 (Google)
- Tailwind CSS color system
- Ant Design color algorithms
- WCAG accessibility guidelines (W3C)
- Color theory from leading UX/UI design resources

### Curated Palettes
All palette examples are inspired by real-world successful products and validated against industry best practices.

## Changelog

### Version 1.0.0 (2026-01-08)
- Initial release
- 60+ curated domain-specific palettes
- WCAG contrast validation script
- Color scale generation script
- Comprehensive reference documentation
- Anti-AI design guidance

## Support

For issues, questions, or suggestions:
- Open an issue on GitHub
- Refer to the comprehensive SKILL.md for detailed usage
- Check references/color-palette-ui-design-reference.md for theory and examples

## Acknowledgments

Created to address the common problem of generic AI-generated color palettes that lack cultural context and professional polish. This skill empowers Claude to create distinctive, accessible, and contextually appropriate color systems.
