# 时间轴图 (Timeline)

在临床病例报告 (Case Report) 中，使用高颜值的时间轴 (Timeline) 梳理患者的诊疗脉络，不仅能让审稿人一目了然，还能大幅提升文章的专业度。

下面是俺使用 R 语言可视化包 `ggplot2` 绘制临床时间轴图的代码。

```r
# 设置英文时间格式（防止月份显示为中文，不符合英文期刊要求）
Sys.setlocale("LC_TIME", "en_US.UTF-8")

# 注册字体（针对 Windows 系统，防止后续强制使用 Arial 字体时报错）
windowsFonts(Arial = windowsFont("Arial"))

# 需要的R包
required_packages <- c(
  "ggplot2",    # 核心绘图包
  "dplyr",      # 数据处理
  "lubridate",  # 时间日期处理
  "scales"      # 坐标轴刻度处理
)

# 未安装则自动下载安装，已安装则直接载入
for (pkg in required_packages) {
  if (!require(pkg, character.only = TRUE, quietly = TRUE)) {
    message(paste("正在安装缺失的R包:", pkg))
    install.packages(pkg, dependencies = TRUE)
    library(pkg, character.only = TRUE)
  }
}

# 录入临床数据(代码中是俺虚构的临床数据)
clinical_data <- data.frame(
  date = ymd(c("2024-02-10", "2024-02-25", "2024-03-15", "2024-08-10", "2024-08-20", "2024-09-15", "2025-03-15")),
  
  # 注意：\n 为手动换行符，保证排版紧凑美观
  event = c("Routine Screening:\nImaging revealed\nsuspicious local mass", 
            "Pathology Confirmation:\nCore biopsy confirmed\ninvasive carcinoma (HR+/HER2-)", 
            "Neoadjuvant Therapy:\nSystemic chemotherapy\ninitiated prior to surgery", 
            "Surgical Intervention:\nTumor resection &\nlymph node evaluation", 
            "Post-op Pathology:\nAssessment showed pathologic\ncomplete response (pCR)", 
            "Adjuvant Treatment:\nRadiotherapy combined\nwith endocrine therapy",
            "Routine Follow-up:\n6-month assessment shows\nno evidence of disease (NED)"),
  
  # 设定事件的类别，便于后续按类别上色
  category = factor(c("Clinical Presentation", "Diagnostic Evaluation", 
                      "Therapeutic Intervention", "Therapeutic Intervention", 
                      "Diagnostic Evaluation", "Therapeutic Intervention",
                      "Follow-up & Outcomes"), 
                    levels = c("Clinical Presentation", "Diagnostic Evaluation", 
                               "Therapeutic Intervention", "Follow-up & Outcomes"))
)

# 自动生成带有时间标签的文本（例如：[Feb 2024] 加上具体事件）
clinical_data$event_label <- paste0("[", format(clinical_data$date, "%b %Y"), "]\n", clinical_data$event)

# 正数代表在时间轴上方，负数代表下方。数值大小代表距离主轴的远近。

# 注意：这里的数字个数必须与你录入的事件总数（本例为 7 个）严格一致！
height_factor <- c(2.8, -3.0, 1.5, -1.8, 3.5, -3.8, 1.2) 
clinical_data$position <- height_factor[1:nrow(clinical_data)]

# 自动生成背景时间刻度（此处设置为每 2 个月一个刻度）
start_month <- floor_date(min(clinical_data$date) - months(1), "month")
end_month <- ceiling_date(max(clinical_data$date) + months(1), "month")
month_breaks <- seq(start_month, end_month, by = "2 months")
month_df <- data.frame(date = month_breaks)



timeline_plot <- ggplot() +
  
  # 绘制中心主轴
  geom_hline(yintercept = 0, color = "#2C3E50", linewidth = 1.2) +
  
  # 绘制月份刻度短线
  geom_segment(data = month_df, aes(x = date, xend = date, y = -0.1, yend = 0.1), 
               color = "#2C3E50", linewidth = 0.8) +
  
  # 添加月份文本标签
  geom_text(data = month_df, aes(x = date, y = -0.3, label = format(date, "%b %Y")), 
            size = 3.5, family = "Arial", fontface = "bold", color = "#666666") +
  
  # 绘制连接时间轴和事件节点的虚线
  geom_segment(data = clinical_data, aes(x = date, xend = date, y = position, yend = 0, color = category), 
               linetype = "dashed", linewidth = 0.6) +
  
  # 绘制事件节点（带白色描边的实心圆点）
  geom_point(data = clinical_data, aes(x = date, y = 0, fill = category), 
             size = 4.5, shape = 21, color = "white", stroke = 1.2) +
  
  #  添加具体的临床事件描述文字
  geom_text(data = clinical_data, aes(x = date, y = position, label = event_label, color = category), 
            size = 3.3,  
            family = "Arial", 
            fontface = "bold", 
            vjust = ifelse(clinical_data$position > 0, -0.1, 1.1), # 根据在轴的上方或下方调整文字对齐方式
            lineheight = 1.1) +
  
  scale_color_manual(values = c("Clinical Presentation" = "#D55E00", 
                                "Diagnostic Evaluation" = "#E69F00", 
                                "Therapeutic Intervention" = "#0072B2", 
                                "Follow-up & Outcomes" = "#009E73")) +
  scale_fill_manual(values = c("Clinical Presentation" = "#D55E00", 
                               "Diagnostic Evaluation" = "#E69F00", 
                               "Therapeutic Intervention" = "#0072B2", 
                               "Follow-up & Outcomes" = "#009E73")) +
  

  scale_y_continuous(limits = c(-5.0, 5.0)) +
  scale_x_date(expand = expansion(mult = c(0.08, 0.08))) + 
  
  theme_void(base_family = "Arial") +
  theme(
    legend.position = "bottom",
    legend.title = element_blank(),
    legend.text = element_text(size = 11, face = "bold", family = "Arial"),
    plot.margin = margin(t = 30, r = 20, b = 25, l = 20, unit = "pt")
  )


print(timeline_plot)

# 不推荐直接使用 RStudio 右下角的 "Export" 按钮

# 导出为 PDF（矢量图）
ggsave("Clinical_Timeline_Plot.pdf", plot = timeline_plot, 
       width = 12, height = 7, units = "in", 
       device = cairo_pdf) # cairo_pdf 能完美嵌入系统字体，防止审稿人打开时字体错乱

# 导出为 TIFF（为了显示足够清晰，设置为 1000 DPI 并开启 LZW 压缩以减小体积）
ggsave("Clinical_Timeline_Plot2.tiff", plot = timeline_plot, 
       width = 12, height = 7, units = "in", 
       dpi = 1000, compression = "lzw",
       bg = "white")
```

### 套用俺的模板需要注意

{% stepper %}
{% step %}
### 向量长度必须一致

`clinical_data` 数据框中的 `date`、`event`、`category` 向量内的元素个数必须完全相等，否则会直接报错。
{% endstep %}

{% step %}
### 防重叠参数 (`height_factor`)

该向量中的数字个数必须与事件数量一致。如果两个事件日期靠得太近，请分配一高一低（如 `3.5` 和 `1.2`），或一正一负（如 `2.8` 和 `-3.0`）的值，拉开它们的垂直距离。
{% endstep %}

{% step %}
### 不要让文字“跑出”画面

代码中默认的 Y 轴范围是 `scale_y_continuous(limits = c(-5.0, 5.0))`。如果在 `height_factor` 中设置了绝对值超过 5 的高度（比如设定为 6），文字将会被裁剪。应对方法是同步扩大 limits 值，例如改为 `c(-8.0, 8.0)`。
{% endstep %}

{% step %}
### 时间跨度与刻度密度

本例虚构数据的病程跨度约 1 年，横坐标按 `by = "2 months"` 显示很合适。如果患者病程较长，按月显示会导致坐标轴密密麻麻挤成一团黑线，请将 `by` 做出适当修改，如修改为 `"6 months"` 或 `"1 year"`。
{% endstep %}

{% step %}
### 自定义图例名称要“一字不差”

如果修改了 `category` 的名称（例如新增了 `"Radiotherapy"`），**请务必同时修改颜色映射字典**（即 `scale_color_manual` 和 `scale_fill_manual` 函数中等号左侧的名字），大小写必须完全匹配，否则该类别的元素将无法显示颜色。
{% endstep %}
{% endstepper %}
