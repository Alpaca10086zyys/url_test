title: "孙朝业" # 姓名
position: "硕士" # 写硕士或博士
contact: "Suncy@mail.nankai.edu.cn" # 邮箱
description: "消防机器人运动规划与控制" # 研究课题
photo: "/url_test/student/sunchaoye/photo.jpg" # 把wanghai改成自己名字的拼音
item:
- 郑州大学学士 # 改成自己的最高学位

---
import math
import time
import numpy as np
import matplotlib.pyplot as plt

show_animation = True
import matplotlib.pyplot as plt
plt.rcParams['font.sans-serif'] = ['SimHei']  # 指定默认字体
plt.rcParams['axes.unicode_minus'] = False      # 解决保存图像是负号'-'显示为方块的问题

class AStarPlanner:

    def __init__(self, ox, oy, resolution, rr):
        """
        Initialize grid map for a star planning

        ox: x position list of Obstacles [m]
        oy: y position list of Obstacles [m]
        resolution: grid resolution [m]
        rr: robot radius[m]
        """

        self.resolution = resolution
        self.rr = rr
        self.min_x, self.min_y = 0, 0
        self.max_x, self.max_y = 0, 0
        self.obstacle_map = None
        self.x_width, self.y_width = 0, 0
        self.motion = self.get_motion_model()
        self.calc_obstacle_map(ox, oy)

    class Node:
        def __init__(self, x, y, cost, parent_index):
            self.x = x  # index of grid
            self.y = y  # index of grid
            self.cost = cost
            self.parent_index = parent_index

        def __str__(self):
            return str(self.x) + "," + str(self.y) + "," + str(
                self.cost) + "," + str(self.parent_index)

    def planning(self, sx, sy, gx, gy):
        """
        A star path search

        input:
            s_x: start x position [m]
            s_y: start y position [m]
            gx: goal x position [m]
            gy: goal y position [m]

        output:
            rx: x position list of the final path
            ry: y position list of the final path
        """

        start_node = self.Node(self.calc_xy_index(sx, self.min_x),
                               self.calc_xy_index(sy, self.min_y), 0.0, -1)
        goal_node = self.Node(self.calc_xy_index(gx, self.min_x),
                              self.calc_xy_index(gy, self.min_y), 0.0, -1)

        open_set, closed_set = dict(), dict()
        open_set[self.calc_grid_index(start_node)] = start_node

        while 1:
            if len(open_set) == 0:
                print("Open set is empty..")
                break

            # 选择扩展点 f(n) = g(n) + h(n)
            c_id = min(
                open_set,
                key=lambda o: open_set[o].cost + self.calc_heuristic(goal_node,
                                                                     open_set[
                                                                         o]))
            current = open_set[c_id]

            # show graph
            if show_animation:  # pragma: no cover
                # plt.plot(self.calc_grid_position(current.x, self.min_x),
                #          self.calc_grid_position(current.y, self.min_y), "xy")
                # for stopping simulation with the esc key.
                plt.gcf().canvas.mpl_connect('key_release_event',
                                             lambda event: [exit(
                                                 0) if event.key == 'escape' else None])
                if len(closed_set.keys()) % 10 == 0:
                    plt.pause(0.001)

            if current.x == goal_node.x and current.y == goal_node.y:
                print("Find goal")
                goal_node.parent_index = current.parent_index
                goal_node.cost = current.cost
                break

            # Remove the item from the open set
            del open_set[c_id]

            # Add it to the closed set
            closed_set[c_id] = current

            # expand_grid search grid based on motion model
            for i, _ in enumerate(self.motion):
                node = self.Node(current.x + self.motion[i][0],
                                 current.y + self.motion[i][1],
                                 current.cost + self.motion[i][2], c_id)
                n_id = self.calc_grid_index(node)

                # If the node is not safe, do nothing
                if not self.verify_node(node):
                    continue

                if n_id in closed_set:
                    continue

                if n_id not in open_set:
                    open_set[n_id] = node  # discovered a new node
                else:
                    if open_set[n_id].cost > node.cost:
                        # This path is the best until now. record it
                        open_set[n_id] = node

        rx, ry = self.calc_final_path(goal_node, closed_set)

        return rx, ry

    def calc_final_path(self, goal_node, closed_set):
        # generate final course
        rx, ry = [self.calc_grid_position(goal_node.x, self.min_x)], [
            self.calc_grid_position(goal_node.y, self.min_y)]
        parent_index = goal_node.parent_index
        while parent_index != -1:
            n = closed_set[parent_index]
            rx.append(self.calc_grid_position(n.x, self.min_x))
            ry.append(self.calc_grid_position(n.y, self.min_y))
            parent_index = n.parent_index

        return rx, ry

    @staticmethod
    def calc_heuristic(n1, n2):
        d = abs(n1.x - n2.x) + abs(n1.y - n2.y)     #  Manhattan distance

        # dx = abs(n1.x - n2.x)                          #  Chebyshev distance 
        # dy = abs(n1.y - n2.y)
        # min_xy = min(dx,dy)
        # d = dx + dy + (math.sqrt(2) - 2) * min_xy   

        # d = math.hypot(n1.x - n2.x, n1.y - n2.y)            #  Euclidean distance
        return d

    def calc_grid_position(self, index, min_position):
        """
        calc grid position

        :param index:
        :param min_position:
        :return:
        """
        pos = index * self.resolution + min_position
        return pos

    def calc_xy_index(self, position, min_pos):
        return round((position - min_pos) / self.resolution)

    def calc_grid_index(self, node):
        return node.y * self.x_width + node.x

    def verify_node(self, node):
        px = self.calc_grid_position(node.x, self.min_x)
        py = self.calc_grid_position(node.y, self.min_y)

        if px < self.min_x:
            return False
        elif py < self.min_y:
            return False
        elif px >= self.max_x:
            return False
        elif py >= self.max_y:
            return False

        # collision check
        if self.obstacle_map[node.x][node.y]:
            return False

        return True

    def calc_obstacle_map(self, ox, oy):

        self.min_x = round(min(ox))
        self.min_y = round(min(oy))
        self.max_x = round(max(ox))
        self.max_y = round(max(oy))
        print("min_x:", self.min_x)
        print("min_y:", self.min_y)
        print("max_x:", self.max_x)
        print("max_y:", self.max_y)

        self.x_width = round((self.max_x - self.min_x) / self.resolution)
        self.y_width = round((self.max_y - self.min_y) / self.resolution)
        print("x_width:", self.x_width)
        print("y_width:", self.y_width)

        # obstacle map generation
        self.obstacle_map = [[False for _ in range(self.y_width)]
                             for _ in range(self.x_width)]
        for ix in range(self.x_width):
            x = self.calc_grid_position(ix, self.min_x)
            for iy in range(self.y_width):
                y = self.calc_grid_position(iy, self.min_y)
                for iox, ioy in zip(ox, oy):
                    d = math.hypot(iox - x, ioy - y)
                    if d <= self.rr:
                        self.obstacle_map[ix][iy] = True
                        break

    @staticmethod
    def get_motion_model():
        # dx, dy, cost
        motion = [[1, 0, 1],
                  [0, 1, 1],
                  [-1, 0, 1],
                  [0, -1, 1],
                  [-1, -1, math.sqrt(2)],
                  [-1, 1, math.sqrt(2)],
                  [1, -1, math.sqrt(2)],
                  [1, 1, math.sqrt(2)]]

        return motion


def main():
    # start and goal position
    sx = 8.0  # [m]
    sy = 4.0  # [m]
    gx = 160.0  # [m]
    gy = 80.0  # [m]
    grid_size = 2.0  # [m]#栅格大小
    robot_radius = 1.5  # [m]#机器人半径

    plt.figure()
    plt.xlabel('x（米）')
    plt.ylabel('y（米）')

    # set obstacle positions
    ox, oy = [], []
    for i in range(180, 228):
        ox.append(i)
        oy.append(0.0)
    for i in range(0,39):
        ox.append(i)
        oy.append(16.0)
    for i in range(44, 119):
        ox.append(i)
        oy.append(16.0)
    for i in range(124, 190):
        ox.append(i)
        oy.append(16.0)
    for i in range(195, 229):
        ox.append(i)
        oy.append(16.0)
    for i in range(0, 39):
        ox.append(i)
        oy.append(32.0)
    for i in range(44, 119):
        ox.append(i)
        oy.append(32.0)
    for i in range(124, 151):
        ox.append(i)
        oy.append(32.0)
    for i in range(151, 163):
        ox.append(i)
        oy.append(32.0)
    for i in range(169, 190):
        ox.append(i)
        oy.append(32.0)
    for i in range(195, 243):
        ox.append(i)
        oy.append(32.0)
    for i in range(83, 119):
        ox.append(i)
        oy.append(48.0)
    for i in range(124, 163):
        ox.append(i)
        oy.append(48.0)
    for i in range(169, 180):
        ox.append(i)
        oy.append(48.0)
    for i in range(0,12):
        ox.append(i)
        oy.append(65.0)
    for i in range(17, 119):
        ox.append(i)
        oy.append(65.0)
    for i in range(124, 163):
        ox.append(i)
        oy.append(65.0)
    for i in range(168,180):
        ox.append(i)
        oy.append(65.0)
    for i in range(180, 244):
        ox.append(i)
        oy.append(93.0)
    for i in range(0, 29):
        ox.append(i)
        oy.append(128.0)
    for i in range(83, 181):
        ox.append(i)
        oy.append(128.0)
    for i in range(16, 128):
        ox.append(0.0)
        oy.append(i)
    for i in range(65, 128):
        ox.append(28.0)
        oy.append(i)
    for i in range(16, 21):
        ox.append(83.0)
        oy.append(i)
    for i in range(26, 37):
        ox.append(83.0)
        oy.append(i)
    for i in range(42, 53):
        ox.append(83.0)
        oy.append(i)
    for i in range(58, 128):
        ox.append(83.0)
        oy.append(i)
    for i in range(32, 37):
        ox.append(151.0)
        oy.append(i)
    for i in range(42, 53):
        ox.append(151.0)
        oy.append(i)
    for i in range(58, 91):
        ox.append(151.0)
        oy.append(i)
    for i in range(96, 128):
        ox.append(151.0)
        oy.append(i)
    for i in range(32, 37):
        ox.append(151.0)
        oy.append(i)
    for i in range(42, 53):
        ox.append(151.0)
        oy.append(i)
    for i in range(0, 21):
        ox.append(180.0)
        oy.append(i)
    for i in range(26, 37):
        ox.append(180.0)
        oy.append(i)
    for i in range(42, 53):
        ox.append(180.0)
        oy.append(i)
    for i in range(58, 79):
        ox.append(180.0)
        oy.append(i)
    for i in range(84, 128):
        ox.append(180.0)
        oy.append(i)
    for i in range(16, 32):
        ox.append(205.0)
        oy.append(i)
    for i in range(32, 60):
        ox.append(214.0)
        oy.append(i)
    for i in range(65, 93):
        ox.append(214.0)
        oy.append(i)
    for i in range(0, 16):
        ox.append(228.0)
        oy.append(i)
    for i in range(32, 93):
        ox.append(243.0)
        oy.append(i)
    for i in range(10, 15):
        for j in range(40, 47):
            ox.append(i)
            oy.append(j)
    for i in range(40, 46):
        for j in range(45, 52):
            ox.append(i)
            oy.append(j)
    if show_animation:  # pragma: no cover
        plt.plot(ox, oy, ".k")
        plt.plot(sx, sy, "og")
        plt.plot(gx, gy, "xr")
        plt.grid(True)
        plt.axis("equal")
        # plt.title("Dynamic weighting_Map2")
    
    start_time = time.time()
    a_star = AStarPlanner(ox, oy, grid_size, robot_radius)
    rx, ry = a_star.planning(sx, sy, gx, gy)
    end_time = time.time()
    all_time = end_time - start_time
    print("The program running time is:",all_time,"s")

    if show_animation:  # pragma: no cover
        plt.plot(rx, ry, "-b")
        plt.pause(0.001)
        plt.show()


if __name__ == '__main__':
    main()
