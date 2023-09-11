#!/usr/bin/env python3

import argparse
import math
from transformers import ThreeDTransformer, TwoDTransformer, Coordinates
from fileIO import FileIO       

class Clip(Coordinates):
    def __init__(self, lines, args, twoDtransformer: TwoDTransformer, d, is_perspective):
        super().__init__()
        self.is_perspective = is_perspective
        self.lines = lines
        self.args = args
        self.twoDtransformer = twoDtransformer
        self.inside = 0 # 0000
        self.left = 1   # 0001
        self.right = 2  # 0010
        self.bottom = 4 # 0100
        self.top = 8    # 1000
        if not self.is_perspective:
            self.x_min = -abs(d)
            self.y_min = -abs(d)
            self.x_max = abs(d)
            self.y_max = abs(d)
        else:
            self.x_min = -1
            self.y_min = -1
            self.x_max = 1
            self.y_max = 1
        self.u_min = self.args.lb_viewportx
        self.v_min = self.args.lb_viewporty
        self.u_max = self.args.ub_viewportx
        self.v_max = self.args.ub_viewporty
        self.d = d

    def world_to_viewport(self, polygons):
        s_x = (self.u_max - self.u_min) / (self.x_max - self.x_min)
        s_y = (self.v_max - self.v_min) / (self.y_max - self.y_min)

        viewport_polygon = []
        for polygon in polygons:
            for line in polygon:
                # print(line)
                if "stroke" in line:
                    viewport_polygon.append(line)
                else:
                    self.twoDtransformer._set_points(line)
                    self.twoDtransformer._translate(self.x_min, self.y_min)
                    self.twoDtransformer._scale(s_x, s_y)
                    self.twoDtransformer._translate(-self.u_min, -self.v_min)
                    viewport_polygon.append([abs(self.twoDtransformer.x1), abs(self.twoDtransformer.y1)])
        return viewport_polygon

    def _set_scan_line_edge(self, y):
        return [self.x_min, y, self.x_max, y]

    def _sort_intersections(self, scan_line_edge, edges, polygon):
        intersections = []
        for edge in edges:
            intersection = self._compute_intersection(polygon[edge[0]], polygon[edge[1]], scan_line_edge)
            intersections.append(intersection[0])
        intersections.sort(key=lambda x: x)
        return intersections

    def _update_parity_bit(self, scan_fill, parity_bit):
        new_parity_bit = parity_bit
        for row in range(len(parity_bit)):
            if scan_fill[row]:
                # grab each pair of fill intersections and fill that range with 1's in that row
                for i in range(0, len(scan_fill[row]), 2):
                    for col in range(math.ceil(scan_fill[row][i]), math.floor(scan_fill[row][i+1])):   
                        try:
                            new_parity_bit[row][col] = 1
                        except IndexError:
                            continue
            else:
                # previous polygon has filled this line
                if 1 not in parity_bit[row]:
                    # fill empty
                    empty = [0]*(len(parity_bit[0]))
                    new_parity_bit[row] = empty
        return new_parity_bit


    def scan_fill(self, polygons):
        organized_polygons = self._prepare_polygons(polygons)
        parity_bit = [[0]*(501) for _ in range(501)]
        for polygon in organized_polygons:
            # returns every y that x amount of edges intersects with it
            scan_fill = self._polygon_scan_fill(polygon)
            # update scan_fill with sorted intersections
            for scan_line, edges in scan_fill.items():
                scan_line_edge = self._set_scan_line_edge(scan_line)
                sorted_intersections = self._sort_intersections(scan_line_edge, edges, polygon)
                # print(scan_line, sorted_intersections)
                scan_fill[scan_line] = sorted_intersections
            parity_bit = self._update_parity_bit(scan_fill, parity_bit)
        return parity_bit
            
    def _polygon_scan_fill(self, polygon):
        scan_line_ys = {}
        for y in range(501):
            edges = []
            for index, line in enumerate(polygon):
                if index+1 < len(polygon) and ("stroke" not in line and "stroke" not in polygon[index+1]):
                    v1 = [line[0], line[1]]
                    v2 = [polygon[index+1][0], polygon[index+1][1]] 
                    y_min = min(v1[1], v2[1])
                    y_max = max(v1[1], v2[1])
                    is_horizontal = y_min - y_max
                    if is_horizontal == 0 or y == y_max:
                        continue
                    if y_min <= y and y < y_max:
                        edges.append([index, index+1])
            scan_line_ys[y] = edges
        return scan_line_ys

    def sutherland_hodgman_clipping(self):
        polygons = self._prepare_polygons(self.lines)
        clipped_polygons = []
        for polygon in polygons:
            # print(polygon)
            clipped_polygons.append(self._single_sutherland_hodgman_clipping(polygon))
        return clipped_polygons

    def _single_sutherland_hodgman_clipping(self, polygon):
        if not self.is_perspective:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    
            edges = self._set_edges(-abs(self.d), -abs(self.d), abs(self.d), abs(self.d))
        else:
            edges = self._set_edges(-1, -1, 1, 1)

        p = polygon
        finalEdgeIndex = 0
        all_outside = 1
        vertices = len(p) - 1
        # [[left], [bottom], [right], [top]]
        for edge in edges:
            clipped_polygon = []
            for index, line in enumerate(p):
                if all_outside == vertices:
                    return []
                elif "stroke" in line:
                    clipped_polygon.append(line)
                    finalEdgeIndex += 1
                elif index+1 < len(p):
                    if "stroke" in p[index+1]:
                        clipped_polygon.append(clipped_polygon[finalEdgeIndex])
                        finalEdgeIndex = index+1
                    else:
                        v1 = [line[0], line[1]]
                        v2 = [p[index+1][0], p[index+1][1]]

                        is_v1_inside = self._is_inside(v1, edge)
                        is_v2_inside = self._is_inside(v2, edge)

                        # both in
                        if is_v1_inside and is_v2_inside:
                            clipped_polygon.append(v2)
                        # both out
                        elif not is_v1_inside and not is_v2_inside:
                            all_outside += 1
                        # v1 in v2 out
                        elif is_v1_inside is True and is_v2_inside is False:
                            inside_outside = self._compute_intersection(v1, v2, edge)
                            clipped_polygon.append(inside_outside)
                        # v1 out v2 in
                        elif is_v1_inside is False and is_v2_inside is True:
                            outside_inside = self._compute_intersection(v1, v2, edge)
                            clipped_polygon.append(outside_inside)
                            clipped_polygon.append(v2)
            p = clipped_polygon
            finalEdgeIndex = 0
        return clipped_polygon

    # sees which lines needs to be clipped and re-calculates coordinates
    def cohen_sutherland_clipping(self):
        clipped_lines = []
        for line in self.lines:
            self._set_points(line)
            x = 0
            y = 0

            p1_code = self._find_code(self.x1, self.y1)
            p2_code = self._find_code(self.x2, self.y2)

            # both in
            if p1_code == 0 and p2_code == 0:
                clipped_lines.append([self.x1, self.y1, self.x2, self.y2, "Line"])
            # both out
            elif (p1_code & p2_code) != 0:
                continue
            else:
                if p1_code == 0:
                    out_code = p2_code
                else:
                    out_code = p1_code

                if out_code == self.left:
                    x = self.x_min
                    y = (self.x_min - self.x1)/(self.x2 - self.x1) * (self.y2 - self.y1) + self.y1
                elif out_code == self.right:
                    x = self.x_max
                    y = (self.x_max - self.x1)/(self.x2 - self.x1) * (self.y2 - self.y1) + self.y1
                elif out_code == self.bottom:
                    y = self.y_min
                    x = (self.y_min - self.y1)/(self.y2 - self.y1) * (self.x2 - self.x1) + self.x1
                elif out_code == self.top:
                    y = self.y_max
                    x = (self.y_max - self.y1)/(self.y2 - self.y1) * (self.x2 - self.x1) + self.x1

                if out_code == p1_code:
                    self.x1 = x
                    self.y1 = y 
                else:
                    self.x2 = x
                    self.y2 = y 
                clipped_lines.append([self.x1, self.y1, self.x2, self.y2, "Line"])
        return clipped_lines

    # calculates the binary value of clipped lines
    def _find_code(self, x, y):
        code = self.inside
        if x < self.x_min:
            code |= self.left
        elif x > self.x_max:
            code |= self.right
        if y < self.y_min:
            code |= self.bottom
        elif y > self.y_max:
            code |= self.top
        return code

    # vertex points: 
    #       x = vertex[0]
    #       y = vertex[1]
    # edge points:
    #   A:
    #       x = edge[0]
    #       y = edge[1]  
    #   B:
    #       x = edge[2]
    #       y = edge[3]
    # vertices are given in counter-clockwise order so "inside" is on the left of the edge
    def _is_inside(self, vertex, edge):
        c = (vertex[0] - edge[0]) * (edge[3] - edge[1]) - (vertex[1] - edge[1]) * (edge[2] - edge[0])
        if "right" in edge or "bottom" in edge:
            c *= -1
        # in/left of line
        if c > 0:
            return True
        # out/right of line
        if c < 0:
            return False
        # on edge
        if c == 0:
            return False

    def _compute_intersection(self, v1, v2, edge):
        new_vertex = []
        x1, y1 = v1[0], v1[1]
        x2, y2 = v2[0], v2[1]
        x3, y3 = edge[0], edge[1]
        x4, y4 = edge[2], edge[3]

        x = ( (x1*y2 - y1*x2) * (x3 - x4) - (x1 - x2) * (x3*y4 - y3*x4) ) / ( (x1 - x2) * (y3 - y4) - (y1 - y2) * (x3 - x4) )
        y = ( (x1*y2 - y1*x2) * (y3 - y4) - (y1 - y2) * (x3*y4 - y3*x4) ) / ( (x1 - x2) * (y3 - y4) - (y1 - y2) * (x3 - x4) )
        new_vertex.append(x)
        new_vertex.append(y)

        return new_vertex

    def _set_edges(self, x_min, y_min, x_max, y_max):
        edges = []
        # right
        edges.append([x_max, y_min, x_max, y_max, "right"])
        # bottom
        edges.append([x_min, y_min, x_max, y_min, "bottom"])
        # left
        edges.append([x_min, y_min, x_min, y_max, "left"])
        # top
        edges.append([x_min, y_max, x_max, y_max, "top"])

        return edges

    def _prepare_polygons(self, lines):
        polygons = []
        polygon = []
        for i in lines:
            if "stroke" not in i:
                polygon.append(i)
            else:
                polygon.append(i)
                polygons.append(polygon)
                polygon = []
        return polygons

def hw4(args):
    fileio = FileIO(args.file)
    faces = fileio.read_smf()

    threeDtransformer = ThreeDTransformer(faces, args)
    if args.projection:
        polygon = threeDtransformer.parallel_normalization()
    else: 
        polygon = threeDtransformer.perspective_normalization()
    
    # print('newlines')
    # for i in polygon:
    #     print(i)

    twoDtransformer = TwoDTransformer(polygon, args)
    clipping = Clip(polygon, args, twoDtransformer, threeDtransformer.d, args.projection)
    clipped_polygons = clipping.sutherland_hodgman_clipping()
    viewport_polygons = clipping.world_to_viewport(clipped_polygons)

    fileio.write_ps(viewport_polygons, args)

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("-f", "--file", type=str, default="bound-lo-sphere.smf")

    parser.add_argument("-j", "--lb_viewportx", type=int, default=0)
    parser.add_argument("-k", "--lb_viewporty", type=int, default=0)
    parser.add_argument("-o", "--ub_viewportx", type=int, default=500)
    parser.add_argument("-p", "--ub_viewporty", type=int, default=500)

    parser.add_argument("-x", "--x_prp", type=float, default=0.0)
    parser.add_argument("-y", "--y_prp", type=float, default=0.0)
    parser.add_argument("-z", "--z_prp", type=float, default=1.0)

    parser.add_argument("-X", "--x_vrp", type=float, default=0.0)
    parser.add_argument("-Y", "--y_vrp", type=float, default=0.0)
    parser.add_argument("-Z", "--z_vrp", type=float, default=0.0)

    parser.add_argument("-q", "--x_vpn", type=float, default=0.0)
    parser.add_argument("-r", "--y_vpn", type=float, default=0.0)
    parser.add_argument("-w", "--z_vpn", type=float, default=-1.0)

    parser.add_argument("-Q", "--x_vup", type=float, default=0.0)
    parser.add_argument("-R", "--y_vup", type=float, default=1.0)
    parser.add_argument("-W", "--z_vup", type=float, default=0.0)

    parser.add_argument("-u", "--umin_vrc", type=float, default=-0.7)
    parser.add_argument("-v", "--vmin_vrc", type=float, default=-0.7)
    parser.add_argument("-U", "--umax_vrc", type=float, default=0.7)
    parser.add_argument("-V", "--vmax_vrc", type=float, default=0.7)

    # perspective projection if false (not present), parallel if true (is present)
    parser.add_argument("-P", "--projection", action="store_true")
    
    parser.add_argument("-F", "--front_clipping", type=float, default=0.6)
    parser.add_argument("-B", "--back_clipping", type=float, default=-0.6)

    args = parser.parse_args()

    hw4(args)

if __name__ == "__main__":
    main()