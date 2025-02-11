# issacgym地形设置

[isaacgym(legged_gym)学习 （二）—— 设置环境地形_legged gym-CSDN博客](https://blog.csdn.net/weixin_45315065/article/details/135450929)

修改文件`terrain.py`,以及对应task的`*_robot_config.py`

- `terrain.py`
  
  ```python
      def make_terrain(self, choice, difficulty):
          terrain = terrain_utils.SubTerrain("terrain",
                                              width=self.width_per_env_pixels,
                                              length=self.width_per_env_pixels,
                                              vertical_scale=self.cfg.vertical_scale,
                                              horizontal_scale=self.cfg.horizontal_scale)
          slope = difficulty * 0.4 # 
          step_height = 0.05 + 0.25 * difficulty
          discrete_obstacles_height = 0.05 + difficulty * 0.5
          stepping_stones_size = 1.5 * (1.05 - difficulty)
          stone_distance = 0.05 if difficulty == 0 else 0.1
          gap_size = 1. * difficulty
          pit_depth = 1. * difficulty
          if choice < self.proportions[0]: # 斜坡
              if choice < self.proportions[0] / 2: 
                  slope *= -1
              terrain_utils.pyramid_sloped_terrain(terrain, slope=slope, platform_size=3.)
          elif choice < self.proportions[1]: # 不平的地形
              terrain_utils.pyramid_sloped_terrain(terrain, slope=slope, platform_size=3.)
              terrain_utils.random_uniform_terrain(terrain, min_height=-0.05, max_height=0.05, step=0.005,
                                                      downsampled_scale=0.2)
          elif choice < self.proportions[3]: # 凹台阶
              if choice < self.proportions[2]:# 凸台阶
                  step_height *= -1
              terrain_utils.pyramid_stairs_terrain(terrain, step_width=0.31, step_height=step_height,
                                                      platform_size=3.)
          elif choice < self.proportions[4]: # 规则的起伏不平
              num_rectangles = 20
              rectangle_min_size = 1.
              rectangle_max_size = 2.
              terrain_utils.discrete_obstacles_terrain(terrain, discrete_obstacles_height, rectangle_min_size,
                                                          rectangle_max_size, num_rectangles, platform_size=3.)
          elif choice < self.proportions[5]: # 有间隙的地形
              terrain_utils.stepping_stones_terrain(terrain, stone_size=stepping_stones_size,
                                                      stone_distance=stone_distance, max_height=0., platform_size=4.)
          elif choice < self.proportions[6]: # gap
              gap_terrain(terrain, gap_size=gap_size, platform_size=3.)
          else:
              pit_terrain(terrain, depth=pit_depth, platform_size=4.)
  
          return terrain
  ```

- `*_robot_config.py`
  
  ```python
      class terrain(LeggedRobotCfg.terrain):
          mesh_type = 'trimesh'  # "heightfield" # none, plane, heightfield or trimesh
          horizontal_scale = 0.1  # [m]
          vertical_scale = 0.005  # [m]
          border_size = 25  # [m]
          curriculum = True
          static_friction = 1.0
          dynamic_friction = 1.0
          restitution = 0.
          # rough terrain only:
          measure_heights = True
          include_act_obs_pair_buf = False
          measured_points_x = [-0.8, -0.7, -0.6, -0.5, -0.4, -0.3, -0.2, -0.1, 0., 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7,
                               0.8]  # 1mx1.6m rectangle (without center line)
          measured_points_y = [-0.5, -0.4, -0.3, -0.2, -0.1, 0., 0.1, 0.2, 0.3, 0.4, 0.5]
  
          # measured_points_x = [-0.45, -0.3, -0.15, 0, 0.15, 0.3, 0.45, 0.6, 0.75, 0.9, 1.05, 1.2] # 1mx1.6m rectangle (without center line)
          # measured_points_y = [-0.75, -0.6, -0.45, -0.3, -0.15, 0., 0.15, 0.3, 0.45, 0.6, 0.75]
  
          selected = False  # select a unique terrain type and pass all arguments
          terrain_kwargs = None  # Dict of arguments for selected terrain
          max_init_terrain_level = 5  # starting curriculum state
          terrain_length = 8.
          terrain_width = 8.
          num_rows = 10  # number of terrain rows (levels)
          num_cols = 20  # number of terrain cols (types)
          # terrain types: [smooth slope, rough slope, stairs up, stairs down, discrete]
          terrain_proportions = [0.2, 0.2, 0.1, 0.1, 0.4]
          # terrain_proportions = [0.1, 0.2, 0.30, 0.30, 0.1]
          # trimesh only:
          slope_treshold = 0.75  # slopes above this threshold will be corrected to vertical surfaces
  ```

或者在`play.py`中也有一定的设置

# 奖惩？
