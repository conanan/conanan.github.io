H:\Tables\_06Nov2025 文件夹中共包含 110 个以 “All\_Subjects\_” 为前缀、“06Nov2025” 为时间标识的 CSV 文件，均为阿尔茨海默病（AD）患者的研究数据。这些文件覆盖 AD 临床研究的核心维度，从患者基础信息到疾病病理特征、从影像评估到分子标志物检测，形成了完整的 AD 研究数据集，可为疾病机制分析、诊断模型构建及治疗效果评估提供支撑，具体分类及内容说明如下：

## 一、认知功能与临床评估类文件（核心疾病表型数据）

此类文件记录 AD 患者认知功能、痴呆严重程度及日常活动能力等关键临床表型，是判断疾病进展阶段的核心依据：



- **All\_Subjects\_ADAS\_06Nov2025.csv**：存储阿尔茨海默病评估量表（ADAS）数据，涵盖记忆、语言、执行功能等多维度认知评估结果，量表得分越高提示认知损害越严重；
- **All\_Subjects\_MMSE\_06Nov2025.csv**：包含简易精神状态检查（MMSE）结果，通过定向力、记忆力、计算力等项目快速筛查认知障碍，总分 30 分，≤27 分提示存在认知异常；
- **All\_Subjects\_MOCA\_06Nov2025.csv**：记录蒙特利尔认知评估（MOCA）数据，相较于 MMSE 更敏感于轻度认知损害（MCI），总分 30 分，≤26 分提示认知异常；
- **All\_Subjects\_CDR\_06Nov2025.csv**：存储临床痴呆评定量表（CDR）结果，通过记忆、定向、判断等领域评估痴呆严重程度，分级为 0（正常）、0.5（可疑痴呆）、1（轻度）、2（中度）、3（重度）；
- **ECOG 系列文件**（如 All\_Subjects\_ECOG12PT\_06Nov2025.csv、All\_Subjects\_ECOGSP\_06Nov2025.csv）：记录日常生活能力量表（ECOG）数据，评估患者穿衣、进食、行走等独立生活能力，反映疾病对患者功能的影响；
- **NPI/NPIQ 系列文件**（All\_Subjects\_NPI\_06Nov2025.csv、All\_Subjects\_NPIQ\_06Nov2025.csv）：包含神经精神症状问卷（NPI）及问卷版（NPIQ）数据，记录患者妄想、幻觉、激越、抑郁等精神行为症状，是 AD 非认知症状评估的重要依据。

## 二、影像检查类文件（疾病病理特征可视化数据）

此类文件涵盖 AD 患者脑部结构、功能及分子影像信息，可直观反映脑内病理改变（如脑萎缩、淀粉样蛋白沉积）：



- **结构性 MRI 相关文件**：


- All\_Subjects\_Structural\_MRI\_Images\_06Nov2025.csv：存储结构性磁共振成像（MRI）影像基础信息，包括扫描参数、脑区体积测量结果（如海马体积、内嗅皮质体积），海马萎缩是 AD 早期典型影像特征；
- All\_Subjects\_Key\_MRI\_06Nov2025.csv：提取 MRI 关键量化指标，如全脑萎缩率、脑室扩大程度等，用于快速评估脑结构异常；
- All\_Subjects\_MRIMETA\_06Nov2025.csv：包含 MRI 影像元数据，如扫描设备型号、序列参数、图像质量评分等，为影像数据标准化及质量控制提供依据。
- **功能性与弥散 MRI 相关文件**：


- All\_Subjects\_Functional\_MRI\_Images\_06Nov2025.csv：记录功能性 MRI（fMRI）数据，反映脑区活动模式（如默认网络功能连接强度），AD 患者常出现默认网络连接异常；
- All\_Subjects\_Diffusion\_MRI\_Images\_06Nov2025.csv、All\_Subjects\_DTIROI\_MEAN\_06Nov2025.csv：存储弥散加权 MRI 及弥散张量成像（DTI）数据，包括各向异性分数（FA）、平均弥散系数（MD）等指标，用于评估脑白质微结构完整性，AD 患者常出现白质纤维束损伤。
- **PET 相关文件**：


- All\_Subjects\_PET\_Images\_06Nov2025.csv、All\_Subjects\_Key\_PET\_06Nov2025.csv：包含正电子发射断层扫描（PET）影像信息，重点记录淀粉样蛋白（Aβ）PET、tau 蛋白 PET 的显像结果，Aβ 沉积（“淀粉样蛋白瀑布” 假说核心）和 tau 蛋白缠结是 AD 特征性病理改变；
- All\_Subjects\_PETQC\_06Nov2025.csv、All\_Subjects\_AV45QC\_06Nov2025.csv：存储 PET 影像质量控制数据，如图像分辨率、噪声水平、示踪剂摄取均匀性等，确保影像数据可靠性；
- All\_Subjects\_PETMETA 系列文件（如 All\_Subjects\_PETMETA\_ADNI1\_06Nov2025.csv）：包含 PET 影像元数据及多中心数据标准化信息，适用于多中心 AD 研究数据整合。
- **病理与 AI 辅助影像文件**：


- All\_Subjects\_Pathology\_Images\_06Nov2025.csv：存储脑组织切片病理图像信息，如 Aβ 沉积、tau 缠结的病理染色结果（如刚果红染色、抗 tau 抗体免疫组化），是 AD 病理诊断金标准；
- All\_Subjects\_MRI\_Images\_with\_AI\_06Nov2025.csv：包含经人工智能（AI）分析的 MRI 影像结果，如 AI 自动分割的脑区体积、AI 辅助检测的微小结构异常，提升影像分析效率与准确性。

## 三、生物标志物类文件（分子水平疾病特征数据）

此类文件记录 AD 患者血液、脑脊液等体液中的分子标志物，是 AD 早期诊断及病理机制研究的关键依据：



- **淀粉样蛋白与 tau 蛋白标志物文件**：


- All\_Subjects\_AMY 系列文件（如 All\_Subjects\_AMYDISC\_06Nov2025.csv、All\_Subjects\_AMYMETA\_06Nov2025.csv）：存储淀粉样蛋白（Aβ）检测数据，包括脑脊液 Aβ42 浓度、Aβ42/Aβ40 比值，血液 Aβ 标志物定量结果，Aβ42/Aβ40 比值降低是 AD 早期分子特征；
- All\_Subjects\_TAU 系列文件（如 All\_Subjects\_TAUMETA\_06Nov2025.csv、All\_Subjects\_JANSSEN\_PLASMA\_P217\_TAU\_06Nov2025.csv）：包含 tau 蛋白标志物数据，如脑脊液总 tau（t-tau）、磷酸化 tau（p-tau181/p-tau217）浓度，血液 P217-tau 浓度，tau 蛋白水平升高反映神经变性程度；
- All\_Subjects\_UCBERKELEY 系列文件（如 All\_Subjects\_UCBERKELEY\_AMY\_6MM\_06Nov2025.csv、All\_Subjects\_UCBERKELEY\_TAU\_6MM\_06Nov2025.csv）：存储经伯克利加州大学标准化处理的 Aβ、tau 标志物数据，包括 PET 显像定量分析（如 6mm 体素水平的示踪剂摄取量）。
- **综合生物标志物与实验室文件**：


- All\_Subjects\_BIOMARK\_06Nov2025.csv：整合多类型生物标志物数据，如 Aβ、tau、炎症因子（如 IL-6、TNF-α）、神经丝轻链蛋白（NfL）等，用于多标志物联合诊断模型构建；
- All\_Subjects\_UPENNBIOMK\_ROCHE\_ELECSYS\_06Nov2025.csv：存储基于罗氏 Elecsys 平台检测的血液生物标志物结果，该平台是临床常用的自动化检测系统，数据可重复性高；
- All\_Subjects\_FNIHBC\_BLOOD\_BIOMARKER\_TRAJECTORIES\_06Nov2025.csv：记录血液生物标志物随时间的变化轨迹（如随访 1 年、2 年的 Aβ、tau 浓度变化），用于评估疾病进展速度及治疗干预效果。

## 四、患者基础信息与临床病史类文件（研究基线与混杂因素数据）

此类文件记录 AD 患者人口学特征、既往病史、家族史等基础信息，为研究数据分层分析及混杂因素控制提供支撑：



- **人口学与研究入组文件**：


- All\_Subjects\_PTDEMOG\_06Nov2025.csv：存储患者人口学数据，包括年龄、性别、教育程度、种族、居住地区（城市 / 农村，对应 All\_Subjects\_RURALITY\_06Nov2025.csv）等，这些因素均可能影响 AD 发病风险及疾病进展；
- All\_Subjects\_Study\_Entry\_06Nov2025.csv、All\_Subjects\_INCLUSIO\_06Nov2025.csv、All\_Subjects\_EXCLUSIO\_06Nov2025.csv：记录患者研究入组信息，包括入组时间、符合的 AD 诊断标准（如 NIA-AA 标准）、入组排除 / 纳入标准执行情况，确保研究样本的同质性。
- **临床病史与用药文件**：


- All\_Subjects\_MEDHIST\_06Nov2025.csv、All\_Subjects\_RECMHIST\_06Nov2025.csv：存储患者既往病史，包括高血压、糖尿病、冠心病等共病情况，这些疾病可能加速 AD 进展；
- All\_Subjects\_BACKMEDS\_06Nov2025.csv、All\_Subjects\_RECCMEDS\_06Nov2025.csv：记录患者既往用药史及当前用药情况，如是否使用胆碱酯酶抑制剂（多奈哌齐）、NMDA 受体拮抗剂（美金刚）等 AD 治疗药物，是否使用抗高血压药、降糖药等共病治疗药物；
- All\_Subjects\_ADVERSE\_06Nov2025.csv：存储患者不良事件记录，如用药副作用（如恶心、头晕）、疾病相关并发症（如跌倒、肺炎），用于治疗安全性评估。
- **家族史与遗传文件**：


- All\_Subjects\_FAMHXPAR\_06Nov2025.csv、All\_Subjects\_FAMHXSIB\_06Nov2025.csv：记录患者家族史，包括父母、兄弟姐妹的 AD 患病情况，AD 具有一定遗传倾向性；
- All\_Subjects\_GENETIC\_06Nov2025.csv、All\_Subjects\_APOERES\_06Nov2025.csv：存储患者遗传学数据，重点记录 APOE 基因型（APOE ε4 等位基因是 AD 重要风险基因），及其他 AD 相关候选基因（如 PSEN1、APP）的突变情况。

## 五、质量控制与辅助评估类文件（数据可靠性与研究合规性保障）

此类文件确保研究数据的可靠性、合规性，同时提供辅助评估信息：



- **质量控制文件**：


- All\_Subjects\_AMYQC\_06Nov2025.csv、All\_Subjects\_PETQC\_06Nov2025.csv、All\_Subjects\_MRINFQ\_06Nov2025.csv：分别针对 Aβ 标志物检测、PET 影像、MRI 影像开展质量控制，记录检测 / 成像过程中的异常情况（如样本污染、图像伪影），并给出质量评分，低质量数据可被排除分析；
- All\_Subjects\_BLSCHECK\_06Nov2025.csv：存储基线数据核查结果，如人口学、病史数据的一致性校验，确保数据录入无误。
- **合规性与知情同意文件**：


- All\_Subjects\_CONSENTS\_06Nov2025.csv、All\_Subjects\_PUBLICITY\_06Nov2025.csv：记录患者知情同意情况，包括是否同意参与研究、是否同意数据用于学术研究、是否同意数据公开共享，符合医学研究伦理规范。
- **辅助评估文件**：


- All\_Subjects\_PHYSICAL\_06Nov2025.csv、All\_Subjects\_VITALS\_06Nov2025.csv：记录患者体格检查结果（如身高、体重、BMI）及生命体征（如血压、心率、体温），反映患者基础健康状态；
- All\_Subjects\_LABDATA\_06Nov2025.csv、All\_Subjects\_LABTESTS\_06Nov2025.csv：存储常规实验室检查数据，如血常规、肝肾功能、电解质等，用于排除其他可能导致认知障碍的疾病（如肝性脑病、电解质紊乱）；
- All\_Subjects\_NEUROEXM\_06Nov2025.csv：记录神经系统体格检查结果，如肌力、肌张力、反射、共济运动等，评估患者神经功能损伤情况。

## 六、文件整体价值与应用场景

该文件夹中的 110 个 CSV 文件形成了 “临床表型 - 影像特征 - 分子标志物 - 基础信息” 四位一体的 AD 研究数据集，可支撑多类研究场景：



1. **AD 诊断模型构建**：整合 MMSE/MOCA 认知量表、海马体积 MRI 指标、血液 p-tau217 等数据，构建早期 AD 诊断模型，提升诊断准确性；
2. **疾病进展预测**：基于 Aβ/tau 标志物变化轨迹、脑萎缩率等纵向数据，预测 AD 患者疾病进展速度，识别高风险进展人群；
3. **治疗效果评估**：在药物临床试验中，通过 ADAS 量表得分变化、PET 影像 Aβ/tau 沉积变化，评估新药（如抗 Aβ 单抗）的治疗效果；
4. **疾病机制研究**：分析 APOE 基因型与 tau 蛋白沉积的关联性、炎症因子与脑白质损伤的相关性，探索 AD 发病机制；
5. **共病分析参考**：对于颈动脉粥样硬化斑块卒中相关研究，可提取 AD 患者中的卒中病史（来自 MEDHIST 文件）、血管风险因素（如高血压、糖尿病）数据，分析 AD 与卒中的共病关联及相互影响。