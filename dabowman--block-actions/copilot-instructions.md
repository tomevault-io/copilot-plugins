## block-actions

> Description: CSS coding standards (formatting, selectors, properties).

Description: CSS coding standards (formatting, selectors, properties).
wordpress_css_standard:
  indentation: "tabs"
  spacing:
    between_sections: 2
    between_blocks: 1
  selector:
    case: "lower"
    separator: "-"
    disallow:
      - camelCase
      - underscores
      - over_qualification
    attribute_quotes: "double"
  declarations:
    format: "property: value;"
    colon_space: true
    semicolon_required: true
    property_case: "lower"
    value_case: "lower"
    color:
      format: ["hex", "rgba"]
      short_hex: true
    zero_unit: false
    leading_zero: true
    line_height_unitless: true
    font_weight_numeric: true
    quotes: "double_only_when_needed"
  property_order:
    strategy: "logical"
    groups: ["display","position","box","color_typography","other"]
    trbl_order: ["top","right","bottom","left"]
    border_radius_order: ["tl","tr","br","bl"]
  vendor_prefix:
    tool: "autoprefixer"
    manual_order: ["-webkit-","-moz-","-o-",""]
  media_queries:
    position: "bottom"
    indent_level: 1
  comments:
    section_format: "phpdoc"
    inline_format: "single_line"
    wrap: 80
  best_practices:
    - "remove_code_before_adding"
    - "avoid_magic_numbers"
    - "target_element_not_parent"
    - "prefer_line_height_over_height"
    - "avoid_redeclaring_defaults"
  resources:
    official: [@https://developer.wordpress.org/coding-standards/wordpress-coding-standards/css/]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dabowman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
