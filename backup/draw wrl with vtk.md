# point
```
import vtk
import re
import numpy as np

# -------------------------- 配置参数（新增点大小设置） --------------------------
VRML_FILE_PATH = r"C:\Users\liuj2\Desktop\10_PDGF x GFP-M#170_Control RH_01x.wrl"
POINT_SIZE = 5.0  # 顶点大小（可调整，越大越清晰）
BACKGROUND_COLOR = (1.0, 1.0, 1.0)  # 白色背景

# -------------------------- 核心工具：解析逻辑不变（仍提取顶点数据） --------------------------
def parse_vrml_manual(vrml_path):
    """手动解析VRML，提取顶点、颜色等数据（解析逻辑不变）"""
    with open(vrml_path, "r", encoding="utf-8") as f:
        content = f.read()
    
    # 正则匹配核心数据（FilamentSegment → 颜色 → 顶点 → 索引）
    pattern = r'''
        DEF\s+(FilamentSegment\d+)\s+Group\s*\{\s*
        .*?diffuseColor\s+(\d+\.?\d*)\s+(\d+\.?\d*)\s+(\d+\.?\d*)\s*
        .*?point\s*\[\s*(.*?)\s*\]
        .*?coordIndex\s*\[\s*(.*?)\s*\]
    '''
    matches = re.findall(pattern, content, re.DOTALL | re.VERBOSE)
    
    if not matches:
        print("⚠️  未匹配到FilamentSegment数据，尝试简化匹配规则...")
        pattern_simple = r'DEF\s+(FilamentSegment\d+)\s+Group\s*\{\s*.*?diffuseColor\s+(\S+)\s+(\S+)\s+(\S+)\s*.*?point\s*\[\s*(.*?)\s*\]'
        matches = re.findall(pattern_simple, content, re.DOTALL)
    
    parsed_data = []
    for match in matches:
        # 解包匹配结果（兼容完整/简化正则）
        if len(match) == 6:
            seg_id, r, g, b, point_str, coord_idx_str = match
        else:
            seg_id, r, g, b, point_str = match
            coord_idx_str = ""
        
        # 解析颜色（转为VTK所需的0-255整数）
        r, g, b = float(r), float(g), float(b)
        if r > 0.7 and g < 0.1 and b > 0.7:
            rgb = (192, 0, 192)  # 树突（紫色）
        elif r > 0.9 and g < 0.1 and b < 0.1:
            rgb = (255, 0, 0)    # 脊柱（红色）
        else:
            rgb = (128, 128, 128)  # 其他结构（灰色）
        
        # 解析顶点坐标（转为(n,3)数组）
        points = re.findall(r'[-+]?\d+\.?\d*e?[-+]?\d*', point_str)
        points = np.array(points, dtype=np.float64).reshape(-1, 3) if points else np.array([])
        
        # 仅保留有顶点的数据（至少1个顶点即可）
        if len(points) >= 1:
            parsed_data.append({
                "seg_id": seg_id,
                "rgb": rgb,
                "points": points
            })
    
    return parsed_data

# -------------------------- 核心逻辑：仅渲染顶点（不连接线段） --------------------------
def vtk_render_points_only(parsed_data):
    """纯顶点渲染，不构建任何线段"""
    vtk_poly_data = vtk.vtkPolyData()
    vtk_points = vtk.vtkPoints()
    vtk_vertices = vtk.vtkCellArray()  # 用于存储顶点（替换原来的vtkLines）
    color_map = vtk.vtkUnsignedCharArray()
    color_map.SetNumberOfComponents(3)
    color_map.SetName("Colors")

    # 遍历解析数据，添加所有顶点（不处理连接关系）
    for seg in parsed_data:
        points = seg["points"]
        rgb = seg["rgb"]

        # 逐个添加顶点和对应的颜色
        for x, y, z in points:
            # 添加顶点坐标
            point_id = vtk_points.InsertNextPoint(x, y, z)
            # 添加顶点颜色
            color_map.InsertNextTuple3(*rgb)
            # 创建单个顶点单元（关键：每个点作为独立单元）
            vertex = vtk.vtkVertex()
            vertex.GetPointIds().SetId(0, point_id)
            vtk_vertices.InsertNextCell(vertex)

    # 组装VTK数据（用顶点单元替换线段单元）
    vtk_poly_data.SetPoints(vtk_points)
    vtk_poly_data.SetVerts(vtk_vertices)  # 渲染顶点（替换原来的SetLines）
    vtk_poly_data.GetPointData().SetScalars(color_map)

    # 配置VTK渲染管线（重点设置点大小）
    mapper = vtk.vtkPolyDataMapper()
    mapper.SetInputData(vtk_poly_data)
    mapper.ScalarVisibilityOn()  # 启用颜色映射

    actor = vtk.vtkActor()
    actor.SetMapper(mapper)
    actor.GetProperty().SetPointSize(POINT_SIZE)  # 设置顶点大小（核心修改）

    # 渲染器配置
    renderer = vtk.vtkRenderer()
    renderer.AddActor(actor)
    renderer.SetBackground(*BACKGROUND_COLOR)
    renderer.ResetCamera()  # 自动调整视角以包含所有点

    # 窗口配置
    render_window = vtk.vtkRenderWindow()
    render_window.SetWindowName("神经细丝顶点可视化（仅显示点）")
    render_window.SetSize(1200, 900)
    render_window.AddRenderer(renderer)

    # 交互器配置
    interactor = vtk.vtkRenderWindowInteractor()
    interactor.SetRenderWindow(render_window)

    # 启动可视化
    interactor.Initialize()
    render_window.Render()
    print("🖱️  交互指南：")
    print("   - 左键拖拽：旋转场景")
    print("   - 滚轮：缩放画面")
    print("   - 右键拖拽：平移场景")
    print("   - 按 'q' 键关闭窗口")
    interactor.Start()

# -------------------------- 主执行流程 --------------------------
# 1. 解析VRML（提取顶点数据）
print("📥 正在解析VRML文件...")
parsed_data = parse_vrml_manual(VRML_FILE_PATH)

if not parsed_data:
    print("❌ 未提取到有效顶点数据！")
else:
    # 统计总顶点数
    total_points = sum(len(seg["points"]) for seg in parsed_data)
    print(f"✅ 成功解析 {len(parsed_data)} 个线段，共 {total_points} 个顶点，启动顶点可视化...")
    # 2. 仅渲染顶点
    vtk_render_points_only(parsed_data)
```



# lines

```import re
import numpy as np
import json
from pathlib import Path
import os

# -------------------------- 配置参数 --------------------------
VRML_FILE_PATH = r"C:\Users\liuj2\Desktop\10_PDGF x GFP-M#170_Control RH_01x.wrl"
EXPORT_FORMAT = "json"  # 导出格式："json"（推荐，保留结构）或 "txt"（易读）
DEFAULT_COLOR = (128, 128, 128)  # 无diffuseColor时的默认颜色

# -------------------------- 核心解析函数（保留所有特征） --------------------------
def parse_vrml_manual(vrml_path):
    """解析VRML，提取所有FilamentSegment的完整特征（适配IndexedFaceSet结构）"""
    with open(vrml_path, "r", encoding="utf-8") as f:
        content = f.read()
    
    # 精准匹配：FilamentSegment → diffuseColor → IndexedFaceSet → Coordinate(point) + coordIndex
    pattern = r'''
        DEF\s+(FilamentSegment\d+)\s+Group\s*\{\s*
        .*?
        (diffuseColor\s+(\d+\.?\d*)\s+(\d+\.?\d*)\s+(\d+\.?\d*)\s*)?  # 颜色（可选）
        .*?
        IndexedFaceSet\s*\{\s*
        .*?
        (coord\s+DEF\s+(\w+)\s+Coordinate\s*\{\s*point\s*\[\s*(.*?)\s*\]\s*\}\s*)  # Coordinate节点（含point）
        .*?
        (normal\s+DEF\s+(\w+)\s+Normal\s*\{\s*vector\s*(.*?)\s*\}\s*)?  # Normal节点（可选，含法向量）
        .*?
        (coordIndex\s*\[\s*(.*?)\s*\])  # coordIndex（面索引）
        .*?
        (normalIndex\s*\[\s*(.*?)\s*\])?  # normalIndex（可选）
        .*?
        (ccw\s+(\w+))?\s*  # ccw参数（可选）
        (solid\s+(\w+))?\s*  # solid参数（可选）
        (convex\s+(\w+))?\s*  # convex参数（可选）
        (creaseAngle\s+(\d+\.?\d*))?\s*  # creaseAngle参数（可选）
        .*?\}\s*  # 关闭IndexedFaceSet
        .*?\}\s*  # 关闭FilamentSegment Group
    '''
    matches = re.findall(pattern, content, re.DOTALL | re.VERBOSE | re.IGNORECASE)
    
    parsed_data = []
    for match in matches:
        # 解包匹配结果（按正则分组顺序）
        (diffuseColor_block, r, g, b,
         coord_block, coord_def_id, point_str,
         normal_block, normal_def_id, normal_vector_str,
         coordIndex_block, coord_idx_str,
         normalIndex_block, normal_idx_str,
         ccw_block, ccw_val,
         solid_block, solid_val,
         convex_block, convex_val,
         creaseAngle_block, creaseAngle_val) = match
        
        # 1. 基础信息（FilamentSegment ID）
        seg_id = re.search(r'FilamentSegment\d+', match[0]).group() if match[0] else "Unknown_Segment"
        
        # 2. 颜色信息
        if diffuseColor_block:
            r, g, b = float(r), float(g), float(b)
            rgb_255 = (int(r*255), int(g*255), int(b*255))
            # 结构类型分类
            if r > 0.7 and g < 0.1 and b > 0.7:
                color_desc = "树突（紫色）"
            elif r > 0.9 and g < 0.1 and b < 0.1:
                color_desc = "脊柱（红色）"
            else:
                color_desc = "其他结构"
        else:
            rgb_255 = DEFAULT_COLOR
            r, g, b = [c/255 for c in DEFAULT_COLOR]
            color_desc = "无颜色信息"
        
        # 3. 顶点坐标（Coordinate → point）
        points_raw = re.findall(r'[-+]?\d+\.?\d*e?[-+]?\d*', point_str)
        points = np.array(points_raw, dtype=np.float64).reshape(-1, 3) if points_raw else np.array([])
        vertex_count = len(points)
        
        # 4. 法向量信息（Normal → vector）
        normal_vector = None
        if normal_block:
            normal_raw = re.findall(r'[-+]?\d+\.?\d*', normal_vector_str)
            normal_vector = tuple(map(float, normal_raw)) if len(normal_raw) == 3 else None
        
        # 5. 面索引（coordIndex）
        coord_idx_groups = []
        if coord_idx_str.strip():
            idx_groups = re.split(r'-1\s*', coord_idx_str.strip())
            for group in idx_groups:
                if group.strip():
                    face_indices = [int(x) for x in group.split()]
                    if len(set(face_indices)) >= 3:  # 过滤无效面
                        coord_idx_groups.append(face_indices)
        face_count = len(coord_idx_groups)
        
        # 6. 其他IndexedFaceSet参数
        ccw = ccw_val.lower() if ccw_val else "true"  # 默认true
        solid = solid_val.lower() if solid_val else "false"  # 默认false
        convex = convex_val.lower() if convex_val else "true"  # 默认true
        crease_angle = float(creaseAngle_val) if creaseAngle_val else 0.0
        
        # 7. normalIndex（可选）
        normal_idx_groups = []
        if normalIndex_block and normal_idx_str.strip():
            idx_groups = re.split(r'-1\s*', normal_idx_str.strip())
            for group in idx_groups:
                if group.strip():
                    normal_idx_groups.append([int(x) for x in group.split()])
        
        # 整理所有特征
        parsed_data.append({
            "基础信息": {
                "FilamentSegment ID": seg_id,
                "结构类型": color_desc,
                "顶点总数": vertex_count,
                "有效面总数": face_count
            },
            "颜色信息": {
                "原始RGB(0-1)": (round(r, 6), round(g, 6), round(b, 6)),
                "标准化RGB(0-255)": rgb_255,
                "颜色描述": color_desc
            },
            "Coordinate节点": {
                "DEF ID": coord_def_id,
                "顶点坐标(point)": points.tolist(),  # 转为list方便导出
                "顶点总数": vertex_count
            },
            "Normal节点": {
                "DEF ID": normal_def_id if normal_def_id else "无",
                "法向量(vector)": normal_vector,
                "是否存在法向量": bool(normal_block)
            },
            "IndexedFaceSet参数": {
                "coordIndex（面索引组）": coord_idx_groups,
                "normalIndex（法向量索引组）": normal_idx_groups if normal_idx_groups else "无",
                "ccw（逆时针排序）": ccw,
                "solid（是否为实心）": solid,
                "convex（是否凸多边形）": convex,
                "creaseAngle（折痕角度）": round(crease_angle, 6)
            }
        })
    
    return parsed_data

# -------------------------- 工具函数：截断长参数显示 --------------------------
def truncate_long_data(data, max_lines=3):
    """长列表/数组显示时仅保留前max_lines行，结尾标注总数"""
    if isinstance(data, list):
        # 处理顶点坐标（每个元素是[x,y,z]）
        if len(data) == 0:
            return "无"
        elif len(data) <= max_lines:
            return [f"[{round(x, 6)}, {round(y, 6)}, {round(z, 6)}]" for x, y, z in data]
        else:
            truncated = [f"[{round(x, 6)}, {round(y, 6)}, {round(z, 6)}]" for x, y, z in data[:max_lines]]
            truncated.append(f"...（共{len(data)}个顶点）")
            return truncated
    elif isinstance(data, list) and all(isinstance(item, list) for item in data):
        # 处理索引组（如coordIndex_groups）
        if len(data) == 0:
            return "无"
        elif len(data) <= max_lines:
            return [str(group) for group in data]
        else:
            truncated = [str(group) for group in data[:max_lines]]
            truncated.append(f"...（共{len(data)}个面索引组）")
            return truncated
    else:
        return data

# -------------------------- 核心功能：导出第一个FilamentSegment --------------------------
def export_first_segment_features(parsed_data):
    if not parsed_data:
        print("❌ 未解析到任何FilamentSegment数据！")
        return
    
    # 仅取第一个FilamentSegment
    first_seg = parsed_data[0]
    seg_id = first_seg["基础信息"]["FilamentSegment ID"]
    print(f"🎉 开始导出第一个FilamentSegment特征：{seg_id}\n")

    # -------------------------- 1. 控制台打印（简化长参数） --------------------------
    print("="*60)
    print("📋 第一个FilamentSegment完整特征（长参数仅显示前3行）")
    print("="*60)
    
    for category, features in first_seg.items():
        print(f"\n【{category}】")
        for key, value in features.items():
            # 对长参数进行截断显示
            if key in ["顶点坐标(point)", "coordIndex（面索引组）"]:
                truncated_val = truncate_long_data(value)
                if isinstance(truncated_val, list):
                    print(f"  {key}:")
                    for line in truncated_val:
                        print(f"    - {line}")
                else:
                    print(f"  {key}: {truncated_val}")
            else:
                print(f"  {key}: {value}")
    
    # -------------------------- 2. 导出完整数据到文件 --------------------------
    # 生成导出路径
    vrml_dir = os.path.dirname(VRML_FILE_PATH)
    vrml_filename = Path(VRML_FILE_PATH).stem
    export_filename = f"{vrml_filename}_第一个FilamentSegment_完整特征.{EXPORT_FORMAT}"
    export_path = os.path.join(vrml_dir, export_filename)
    
    # 导出为JSON（保留完整结构）或TXT（易读）
    if EXPORT_FORMAT == "json":
        # 转换numpy数组为list（JSON不支持numpy类型）
        with open(export_path, "w", encoding="utf-8") as f:
            json.dump(first_seg, f, ensure_ascii=False, indent=2)
    elif EXPORT_FORMAT == "txt":
        with open(export_path, "w", encoding="utf-8") as f:
            f.write(f"第一个FilamentSegment完整特征 - {seg_id}\n")
            f.write("="*80 + "\n\n")
            for category, features in first_seg.items():
                f.write(f"【{category}】\n")
                for key, value in features.items():
                    f.write(f"  {key}: {value}\n")
                f.write("\n")
    
    print(f"\n📁 完整特征已导出至：{export_path}")
    print(f"💡 导出格式：{EXPORT_FORMAT}（含未截断的完整数据）")

# -------------------------- 主函数 --------------------------
if __name__ == "__main__":
    print("📥 正在解析VRML文件...")
    parsed_data = parse_vrml_manual(VRML_FILE_PATH)
    
    if parsed_data:
        print(f"✅ 共解析到 {len(parsed_data)} 个FilamentSegment")
        export_first_segment_features(parsed_data)
    else:
        print("❌ 解析失败：未找到符合要求的FilamentSegment结构！")
```


# faces


```
import vtk
import re
import numpy as np

# -------------------------- 配置参数（适配面渲染，保留核心项） --------------------------
VRML_FILE_PATH = r"C:\Users\liuj2\Desktop\10_PDGF x GFP-M#170_Control RH_01x.wrl"
FACE_OPACITY = 0.7  # 面透明度（0=透明，1=不透明）
EDGE_WIDTH = 1.0    # 面边缘宽度（突出面边界）
EDGE_COLOR = (0.0, 0.0, 0.0)  # 面边缘颜色（黑色）
BACKGROUND_COLOR = (1.0, 1.0, 1.0)  # 白色背景

# -------------------------- 核心工具：解析coordIndex并保留-1作为面分隔符 --------------------------
def parse_vrml_manual(vrml_path):
    """手动解析VRML，保留coordIndex中的-1，提取面数据（ID、颜色、顶点、面索引组）"""
    with open(vrml_path, "r", encoding="utf-8") as f:
        content = f.read()
    
    # 正则匹配核心数据（FilamentSegment → 颜色 → 顶点 → 带-1的coordIndex）
    pattern = r'''
        DEF\s+(FilamentSegment\d+)\s+Group\s*\{\s*
        .*?diffuseColor\s+(\d+\.?\d*)\s+(\d+\.?\d*)\s+(\d+\.?\d*)\s*
        .*?point\s*\[\s*(.*?)\s*\]
        .*?coordIndex\s*\[\s*(.*?)\s*\]
    '''
    matches = re.findall(pattern, content, re.DOTALL | re.VERBOSE)
    
    if not matches:
        print("⚠️  未匹配到FilamentSegment数据，尝试简化匹配规则...")
        pattern_simple = r'DEF\s+(FilamentSegment\d+)\s+Group\s*\{\s*.*?diffuseColor\s+(\S+)\s+(\S+)\s+(\S+)\s*.*?point\s*\[\s*(.*?)\s*\]'
        matches = re.findall(pattern_simple, content, re.DOTALL)
    
    parsed_data = []
    for match in matches:
        # 解包匹配结果（兼容完整/简化正则）
        if len(match) == 6:
            seg_id, r, g, b, point_str, coord_idx_str = match
        else:
            seg_id, r, g, b, point_str = match
            coord_idx_str = ""
        
        # 解析颜色（转为VTK所需的0-255整数）
        r, g, b = float(r), float(g), float(b)
        if r > 0.7 and g < 0.1 and b > 0.7:
            rgb = (192, 0, 192)  # 树突（紫色）
        elif r > 0.9 and g < 0.1 and b < 0.1:
            rgb = (255, 0, 0)    # 脊柱（红色）
        else:
            rgb = (128, 128, 128)  # 其他结构（灰色）
        
        # 解析顶点坐标（转为(n,3)数组）
        points = re.findall(r'[-+]?\d+\.?\d*e?[-+]?\d*', point_str)
        points = np.array(points, dtype=np.float64).reshape(-1, 3) if points else np.array([])
        
        # -------------------------- 关键修改：保留-1，按-1分割面索引 --------------------------
        face_groups = []
        if coord_idx_str.strip():
            # 步骤1：移除逗号（处理"0,1,18,17,-1" → "0 1 18 17 -1"）
            coord_idx_str_clean = coord_idx_str.replace(',', '').strip()
            # 步骤2：按-1分割，得到每个面的索引组（保留-1作为分隔符，分割后丢弃-1）
            idx_groups = re.split(r'-1\s*', coord_idx_str_clean)
            # 步骤3：过滤空组，且每个面需≥3个顶点才有效
            for group in idx_groups:
                if group.strip():
                    face_indices = [int(x) for x in group.split()]
                    if len(face_indices) >= 3:  # 面至少需要3个顶点
                        face_groups.append(face_indices)
        
        # 仅保留有效数据（至少3个顶点 + 至少1个有效面）
        if len(points) >= 3 and len(face_groups) >= 1:
            parsed_data.append({
                "seg_id": seg_id,
                "rgb": rgb,
                "points": points,
                "face_groups": face_groups  # 存储按-1分割后的面索引组
            })
    
    return parsed_data

# -------------------------- 核心逻辑：VTK绘制面（face）而非线（line） --------------------------
def vtk_face_visualization(parsed_data):
    """用VTK绘制3D面，保留面边缘，无冗余功能"""
    vtk_poly_data = vtk.vtkPolyData()
    vtk_points = vtk.vtkPoints()
    vtk_faces = vtk.vtkCellArray()  # 存储面（替换原vtkLines）
    color_map = vtk.vtkUnsignedCharArray()
    color_map.SetNumberOfComponents(3)
    color_map.SetName("Colors")

    # 遍历解析数据，构建VTK顶点和面
    for seg in parsed_data:
        points = seg["points"]
        face_groups = seg["face_groups"]  # 按-1分割后的面索引组
        rgb = seg["rgb"]

        # 添加顶点和颜色（与原逻辑一致）
        start_point_id = vtk_points.GetNumberOfPoints()
        for x, y, z in points:
            vtk_points.InsertNextPoint(x, y, z)
            color_map.InsertNextTuple3(*rgb)

        # -------------------------- 关键修改：用vtkPolygon创建面（替换原vtkLine） --------------------------
        for face_indices in face_groups:
            # 创建单个面（多边形）
            polygon = vtk.vtkPolygon()
            polygon.GetPointIds().SetNumberOfIds(len(face_indices))
            # 映射局部索引到全局顶点ID
            for i, idx in enumerate(face_indices):
                global_point_id = start_point_id + idx
                polygon.GetPointIds().SetId(i, global_point_id)
            # 添加面到面数组
            vtk_faces.InsertNextCell(polygon)

    # 组装VTK数据（用SetPolys替换原SetLines）
    vtk_poly_data.SetPoints(vtk_points)
    vtk_poly_data.SetPolys(vtk_faces)  # 绘制面（核心修改）
    vtk_poly_data.GetPointData().SetScalars(color_map)

    # 配置VTK渲染管线（适配面渲染）
    mapper = vtk.vtkPolyDataMapper()
    mapper.SetInputData(vtk_poly_data)
    mapper.ScalarVisibilityOn()

    actor = vtk.vtkActor()
    actor.SetMapper(mapper)
    # 面渲染属性（透明度+边缘）
    actor.GetProperty().SetOpacity(FACE_OPACITY)
    actor.GetProperty().SetEdgeVisibility(True)  # 显示面边缘
    actor.GetProperty().SetEdgeColor(*EDGE_COLOR)  # 边缘颜色
    actor.GetProperty().SetLineWidth(EDGE_WIDTH)  # 边缘宽度

    # 渲染器配置（与原逻辑一致）
    renderer = vtk.vtkRenderer()
    renderer.AddActor(actor)
    renderer.SetBackground(*BACKGROUND_COLOR)
    renderer.ResetCamera()  # 自动适配所有面

    render_window = vtk.vtkRenderWindow()
    render_window.SetWindowName("神经细丝VTK面可视化")
    render_window.SetSize(1200, 900)
    render_window.AddRenderer(renderer)

    interactor = vtk.vtkRenderWindowInteractor()
    interactor.SetRenderWindow(render_window)

    # 启动可视化交互（与原逻辑一致）
    interactor.Initialize()
    render_window.Render()
    print("🖱️  交互指南：")
    print("   - 左键拖拽：旋转场景")
    print("   - 滚轮：缩放画面")
    print("   - 右键拖拽：平移场景")
    print("   - 按 'q' 键关闭窗口")
    interactor.Start()

# -------------------------- 主函数（纯面可视化流程） --------------------------
if __name__ == "__main__":
    # 1. 解析VRML（保留-1，提取面数据）
    print("📥 正在解析VRML文件...")
    parsed_data = parse_vrml_manual(VRML_FILE_PATH)

    if not parsed_data:
        print("❌ 未提取到有效面数据！")
    else:
        total_segments = len(parsed_data)
        total_faces = sum(len(seg["face_groups"]) for seg in parsed_data)
        total_points = sum(len(seg["points"]) for seg in parsed_data)
        print(f"✅ 成功解析 {total_segments} 个神经细丝段，{total_points} 个顶点，{total_faces} 个面，启动VTK面可视化...")
        # 2. VTK绘制面（无其他功能）
        vtk_face_visualization(parsed_data)
```