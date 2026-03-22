import math
import random
import sys
from array import array
from dataclasses import dataclass

import pygame


SCREEN_WIDTH = 1280
SCREEN_HEIGHT = 720
FPS = 60
MATCH_TIME_SECONDS = 180
AI_UPDATE_INTERVAL = 0.1

PITCH_WIDTH = 760.0
PITCH_LENGTH = 1180.0
GOAL_WIDTH = 252.0
GOAL_DEPTH = 94.0
GOAL_HEIGHT = 110.0
PENALTY_BOX_WIDTH = 390.0
PENALTY_BOX_DEPTH = 190.0
SIX_BOX_WIDTH = 210.0
SIX_BOX_DEPTH = 86.0
CENTER_CIRCLE_RADIUS = 92.0

PROJECT_SCALE = 980.0
HORIZON_Y = 132
CAMERA_HEIGHT = 176.0
CAMERA_BACK_OFFSET = 275.0
CAMERA_X_DAMPING = 4.0
CAMERA_Z_DAMPING = 3.2
NEAR_PLANE = 20.0

BALL_RADIUS = 8.0
BALL_RENDER_SCALE = 0.82
BALL_GRAVITY = 720.0
BALL_AIR_DRAG = 0.18
BALL_GROUND_DRAG = 1.48
BALL_BOUNCE = 0.40
BALL_MIN_BOUNCE_SPEED = 44.0
BALL_CONTROL_RADIUS = 34.0
BALL_CONTROL_HEIGHT = 32.0
BALL_CONTROL_SPEED = 180.0
BALL_TRAIL_LENGTH = 10
PASS_LEAD_FACTOR = 0.24
PASS_MIN_POWER = 240.0
PASS_MAX_POWER = 540.0
PASS_RECEIVER_BONUS = 115.0
INTERCEPT_PENALTY = 60.0
STEAL_DISTANCE = 42.0
STEAL_BASE_CHANCE = 0.08
STEAL_SKILL_FACTOR = 0.18
STEAL_DIFFICULTY_SCALE = 0.58
STEAL_COOLDOWN = 0.46
POSSESSION_PROTECTION_TIME = 0.28
SHOT_CURVE_FORCE = 185.0
SHOT_CURVE_DURATION = 0.86
SHOT_CURVE_DECAY = 1.95
SHOT_CURVE_ZONE_PADDING = 26.0
SHOT_CURVE_ATTACK_PADDING = 118.0
SHOT_CURVE_MAX_DISTANCE = 430.0
SHOT_KEEPER_CURVE_FORCE = 228.0
SHOT_KEEPER_CURVE_DISTANCE = 210.0
SHOT_KEEPER_FINISH_DISTANCE = 255.0
SHOT_KEEPER_TARGET_OFFSET = 84.0
SHOT_STRAIGHT_CURL_CHANCE = 1.0
SHOT_STRAIGHT_CURL_DISTANCE = 525.0
SHOT_STRAIGHT_ON_TARGET_WEIGHT = 0.56
SHOT_STRAIGHT_POST_WEIGHT = 0.24
SHOT_STRAIGHT_OFF_WEIGHT = 0.20
SHOT_STRAIGHT_CURVE_FORCE = 158.0
SHOT_STRAIGHT_POST_FORCE = 182.0
SHOT_STRAIGHT_OFF_FORCE = 148.0
BALL_CURVE_DECAY = 0.972
BALL_CURVE_ACCEL = 6.40
BALL_CURVE_MIN_SPEED = 110.0
BALL_CURVE_VISUAL_BLEND = 0.10
SHORT_SHOT_CHARGE_MAX = 0.38
MID_SHOT_CHARGE_MAX = 0.74
SHORT_SHOT_CURVE_SCALE = 2.10
MID_SHOT_CURVE_SCALE = 3.20
LONG_SHOT_CURVE_SCALE = 4.80
SHORT_SHOT_CURVE_BLEND = 0.10
MID_SHOT_CURVE_BLEND = 0.04
LONG_SHOT_CURVE_BLEND = 0.0
SHORT_SHOT_POWER_SCALE = 1.08
MID_SHOT_POWER_SCALE = 1.20
LONG_SHOT_POWER_SCALE = 1.36
SHORT_SHOT_LIFT_SCALE = 1.00
MID_SHOT_LIFT_SCALE = 1.08
LONG_SHOT_LIFT_SCALE = 1.16
GOALKEEPER_MOVE_SPEED = 148.0
GOALKEEPER_OWNER_SAVE_REACH = 32.0
GOALKEEPER_SHOT_SAVE_REACH = 36.0

PLAYER_HEIGHT = 64.0
GOALKEEPER_HEIGHT = 70.0

REAL_MADRID = "Real Madrid"
BARCELONA = "Barcelona"

SKY_TOP = (46, 84, 148)
SKY_BOTTOM = (169, 196, 228)
GRASS_A = (36, 135, 66)
GRASS_B = (31, 117, 57)
LINE_COLOR = (244, 244, 244)
SHADOW_COLOR = (0, 0, 0, 85)
GOAL_FRAME = (231, 231, 231)
HUD_PANEL = (17, 22, 31, 185)

V2 = pygame.Vector2
V3 = pygame.Vector3

SHOT_NORMAL = 0
SHOT_POWER = 1
SHOT_CURVE = 2
SHOT_KNUCKLE = 3
SHOT_VOLLEY = 4

SHOT_LABELS = {
    SHOT_NORMAL: "NORMAL SHOT",
    SHOT_POWER: "POWER SHOT",
    SHOT_CURVE: "CURVE SHOT",
    SHOT_KNUCKLE: "KNUCKLEBALL",
    SHOT_VOLLEY: "VOLLEY",
}

POWER_SHOT_MIN_CHARGE = 0.56
VOLLEY_TRIGGER_HEIGHT = 8.0
KNUCKLE_DECAY = 0.964
SHOT_AIM_X_OFFSET = 86.0
SHOT_TOP_BINS_LIFT = 110.0
SHOT_LOW_DRIVE_LIFT = 26.0
SHOT_INPUT_CURVE_FORCE = 248.0


@dataclass(frozen=True)
class PlayerStats:
    max_speed: float
    sprint_speed: float
    acceleration: float
    shot_power: float
    shot_lift: float
    ball_control: float
    dribble: float
    curve_power: float
    curve_decay: float
    curve_control: float


RONALDO_STATS = PlayerStats(191.0, 262.0, 620.0, 760.0, 240.0, 0.80, 0.78, 1.12, 0.985, 0.84)
MESSI_STATS = PlayerStats(184.0, 252.0, 700.0, 675.0, 210.0, 0.95, 0.95, 0.82, 0.978, 0.96)
BENZEMA_STATS = PlayerStats(186.0, 250.0, 605.0, 715.0, 220.0, 0.88, 0.84, 0.88, 0.980, 0.87)
SUAREZ_STATS = PlayerStats(187.0, 252.0, 610.0, 705.0, 218.0, 0.86, 0.83, 0.92, 0.979, 0.85)


def clamp(value, minimum, maximum):
    return max(minimum, min(maximum, value))


def lerp(a, b, t):
    return a + (b - a) * t


def point_segment_distance(point, start, end):
    segment = end - start
    length_sq = segment.length_squared()
    if length_sq <= 0.01:
        return point.distance_to(start)
    t = clamp((point - start).dot(segment) / length_sq, 0.0, 1.0)
    projection = start + segment * t
    return point.distance_to(projection)


def create_tone(frequency, duration_ms, volume=0.35, sample_rate=22050):
    sample_count = int(sample_rate * (duration_ms / 1000.0))
    buffer = array("h")
    amplitude = int(32767 * volume)
    for index in range(sample_count):
        sample = int(amplitude * math.sin(2.0 * math.pi * frequency * (index / sample_rate)))
        buffer.append(sample)
    return pygame.mixer.Sound(buffer=buffer.tobytes())


class Player3D:
    def __init__(self, game, name, team_name, stats, attack_direction, start_pos, colors, label):
        self.game = game
        self.name = name
        self.team_name = team_name
        self.stats = stats
        self.attack_direction = attack_direction
        self.spawn_pos = V2(start_pos)
        self.pos = V2(start_pos)
        self.vel = V2()
        self.intent = V2()
        self.sprint = False
        self.facing = V2(0, attack_direction)
        self.colors = colors
        self.label = label
        self.label_surface = game.get_cached_text(game.small_font, label, (255, 255, 255))
        self.height = PLAYER_HEIGHT
        self.kick_cooldown = 0.0
        self.animation_time = 0.0
        self.shoot_flash = 0.0
        self.possession_buffer = 0.0

    @property
    def control_radius(self):
        return BALL_CONTROL_RADIUS + self.stats.ball_control * 8.0

    def reset(self, position):
        self.spawn_pos.update(position)
        self.pos.update(position)
        self.vel.update(0, 0)
        self.intent.update(0, 0)
        self.sprint = False
        self.facing.update(0, self.attack_direction)
        self.kick_cooldown = 0.0
        self.animation_time = 0.0
        self.shoot_flash = 0.0
        self.possession_buffer = 0.0

    def update(self, dt):
        if self.kick_cooldown > 0:
            self.kick_cooldown = max(0.0, self.kick_cooldown - dt)
        if self.shoot_flash > 0:
            self.shoot_flash = max(0.0, self.shoot_flash - dt)
        if self.possession_buffer > 0:
            self.possession_buffer = max(0.0, self.possession_buffer - dt)

        if self.intent.length_squared() > 0.01:
            self.facing = self.intent.normalize()

        if self.game.play_state == "PLAYING":
            target_speed = self.stats.sprint_speed if self.sprint else self.stats.max_speed
            target_velocity = self.intent.normalize() * target_speed if self.intent.length_squared() > 0.01 else V2()
            delta = target_velocity - self.vel
            max_delta = self.stats.acceleration * dt
            if delta.length_squared() > max_delta * max_delta:
                delta.scale_to_length(max_delta)
            self.vel += delta
            self.vel *= 0.90 if self.intent.length_squared() <= 0.01 else 0.985
            self.pos += self.vel * dt
        else:
            self.vel *= 0.8

        self.pos.x = clamp(self.pos.x, -PITCH_WIDTH / 2 + 24, PITCH_WIDTH / 2 - 24)
        self.pos.y = clamp(self.pos.y, 24, PITCH_LENGTH - 24)
        self.animation_time += dt * (1.4 + self.vel.length() / 170.0)

    def can_control_ball(self, ball):
        return (
            ball.pos.y <= BALL_CONTROL_HEIGHT + self.stats.ball_control * 10.0
            and ball.horizontal_speed() <= BALL_CONTROL_SPEED + self.stats.ball_control * 80.0
            and self.pos.distance_to(V2(ball.pos.x, ball.pos.z)) <= self.control_radius
        )

    def get_curve_profile(self, target_goal_center, charge):
        distance_to_goal = self.pos.distance_to(target_goal_center)
        wide_limit = PENALTY_BOX_WIDTH * 0.5 + SHOT_CURVE_ZONE_PADDING
        is_wide_channel = abs(self.pos.x) >= wide_limit
        if self.attack_direction > 0:
            attacking_zone = self.pos.y >= PITCH_LENGTH - PENALTY_BOX_DEPTH - SHOT_CURVE_ATTACK_PADDING
        else:
            attacking_zone = self.pos.y <= PENALTY_BOX_DEPTH + SHOT_CURVE_ATTACK_PADDING

        if not is_wide_channel or not attacking_zone or distance_to_goal > SHOT_CURVE_MAX_DISTANCE:
            return 0.0, 0.0

        curve_target_x = clamp(-self.pos.x * 0.18, -GOAL_WIDTH * 0.22, GOAL_WIDTH * 0.22)
        curve_force = lerp(SHOT_CURVE_FORCE * 0.70, SHOT_CURVE_FORCE, charge)
        return curve_force, curve_target_x

    def get_goalkeeper_curve_profile(self, target_goal_center, charge):
        goalkeeper = self.game.get_target_goalkeeper(self.attack_direction)
        if goalkeeper is None:
            return None

        distance_to_goal = self.pos.distance_to(target_goal_center)
        distance_to_keeper = self.pos.distance_to(goalkeeper.pos)
        if distance_to_goal > SHOT_KEEPER_FINISH_DISTANCE or distance_to_keeper > SHOT_KEEPER_CURVE_DISTANCE:
            return None

        target_offset = -SHOT_KEEPER_TARGET_OFFSET if goalkeeper.pos.x >= 0.0 else SHOT_KEEPER_TARGET_OFFSET
        target_x = clamp(target_offset, -GOAL_WIDTH * 0.34, GOAL_WIDTH * 0.34)
        curve_force = lerp(SHOT_KEEPER_CURVE_FORCE * 0.74, SHOT_KEEPER_CURVE_FORCE, charge)
        return target_x, curve_force

    def get_straight_curve_profile(self, target_goal_center, charge):
        distance_to_goal = self.pos.distance_to(target_goal_center)
        if abs(self.pos.x) > GOAL_WIDTH * 0.42 or distance_to_goal > SHOT_STRAIGHT_CURL_DISTANCE:
            return None

        trigger_chance = lerp(SHOT_STRAIGHT_CURL_CHANCE * 0.80, SHOT_STRAIGHT_CURL_CHANCE, charge)
        if random.random() > trigger_chance:
            return None

        if self.pos.x > 18.0:
            side_sign = -1.0
        elif self.pos.x < -18.0:
            side_sign = 1.0
        else:
            side_sign = random.choice((-1.0, 1.0))
        roll = random.random()
        post_x = side_sign * (GOAL_WIDTH * 0.5 - BALL_RADIUS * 1.1)

        if roll < SHOT_STRAIGHT_ON_TARGET_WEIGHT:
            target_x = side_sign * random.uniform(GOAL_WIDTH * 0.10, GOAL_WIDTH * 0.26)
            curve_force = lerp(SHOT_STRAIGHT_CURVE_FORCE * 0.72, SHOT_STRAIGHT_CURVE_FORCE, charge)
        elif roll < SHOT_STRAIGHT_ON_TARGET_WEIGHT + SHOT_STRAIGHT_POST_WEIGHT:
            target_x = post_x
            curve_force = lerp(SHOT_STRAIGHT_POST_FORCE * 0.74, SHOT_STRAIGHT_POST_FORCE, charge)
        else:
            target_x = side_sign * random.uniform(GOAL_WIDTH * 0.56, GOAL_WIDTH * 0.74)
            curve_force = lerp(SHOT_STRAIGHT_OFF_FORCE * 0.70, SHOT_STRAIGHT_OFF_FORCE, charge)

        return clamp(target_x, -PITCH_WIDTH * 0.42, PITCH_WIDTH * 0.42), curve_force

    def get_shot_direction(self, charge, target_goal_center, target_x_override=None, curve_blend=1.0):
        to_goal = target_goal_center - self.pos
        if to_goal.length_squared() <= 0.01:
            to_goal = V2(0, self.attack_direction)

        direction = to_goal.normalize()
        if self.intent.length_squared() > 0.01:
            direction = self.intent.normalize().lerp(direction, 0.78)

        side_bias = clamp(-self.pos.x * 0.09, -58.0, 58.0)
        if abs(self.pos.x) > GOAL_WIDTH * 0.45 and self.pos.y > PITCH_LENGTH * 0.55:
            side_bias += 36.0 if self.pos.x > 0 else -36.0

        if target_x_override is None:
            target_x = target_goal_center.x + side_bias
        else:
            base_target_x = target_goal_center.x
            target_x = lerp(base_target_x, target_x_override, clamp(curve_blend, 0.0, 1.0))
        target = V2(target_x, target_goal_center.y)
        shot_vector = target - self.pos
        if shot_vector.length_squared() <= 0.01:
            shot_vector = V2(0, self.attack_direction)
        return shot_vector.normalize()

    def get_shot_curve_profile(self, charge):
        if charge <= SHORT_SHOT_CHARGE_MAX:
            return SHORT_SHOT_CURVE_SCALE, SHORT_SHOT_CURVE_BLEND
        if charge <= MID_SHOT_CHARGE_MAX:
            return MID_SHOT_CURVE_SCALE, MID_SHOT_CURVE_BLEND
        return LONG_SHOT_CURVE_SCALE, LONG_SHOT_CURVE_BLEND

    def get_shot_power_profile(self, charge):
        if charge <= SHORT_SHOT_CHARGE_MAX:
            return SHORT_SHOT_POWER_SCALE, SHORT_SHOT_LIFT_SCALE
        if charge <= MID_SHOT_CHARGE_MAX:
            return MID_SHOT_POWER_SCALE, MID_SHOT_LIFT_SCALE
        return LONG_SHOT_POWER_SCALE, LONG_SHOT_LIFT_SCALE

    def resolve_shot_type(self, requested_type, charge, ball):
        if ball.owner is not self and ball.pos.y > VOLLEY_TRIGGER_HEIGHT:
            return SHOT_VOLLEY
        if requested_type == SHOT_NORMAL and charge >= POWER_SHOT_MIN_CHARGE:
            return SHOT_POWER
        return requested_type

    def get_open_goal_side(self):
        goalkeeper = self.game.get_target_goalkeeper(self.attack_direction)
        if goalkeeper is None:
            return -1.0 if self.pos.x > 0.0 else 1.0
        return -1.0 if goalkeeper.pos.x >= 0.0 else 1.0

    def get_manual_shot_input(self, shot_type, power_ratio):
        aim_x = self.game.shot_aim_x * SHOT_AIM_X_OFFSET
        lift_bonus = self.game.shot_lift_input * SHOT_TOP_BINS_LIFT * power_ratio
        if self.game.shot_lift_input < 0.0:
            lift_bonus = self.game.shot_lift_input * SHOT_LOW_DRIVE_LIFT

        curve_input = self.game.shot_curve_input
        if abs(curve_input) < 0.01:
            curve_input = self.game.shot_aim_x * 0.65
        if shot_type == SHOT_POWER:
            curve_input *= 0.80
        elif shot_type == SHOT_KNUCKLE:
            curve_input = 0.0
        elif shot_type == SHOT_CURVE:
            curve_input *= 1.20
        return aim_x, lift_bonus, clamp(curve_input, -1.0, 1.0)

    def get_player_shot_profile(self, shot_type):
        accuracy = clamp(0.68 + self.stats.ball_control * 0.15 + self.stats.curve_control * 0.12, 0.68, 0.92)
        profile = {
            "power_bonus": 1.0,
            "curve_scale": 0.70,
            "lift_bonus": 1.0,
            "accuracy": accuracy,
            "post_chance": 0.08,
            "top_bins_chance": 0.18,
            "curve_blend": 0.18,
            "knuckle_strength": 0.0,
        }

        if shot_type == SHOT_POWER:
            profile.update(
                power_bonus=1.34,
                curve_scale=0.60,
                lift_bonus=1.20,
                accuracy=accuracy - 0.08,
                post_chance=0.08,
                top_bins_chance=0.35,
                curve_blend=0.20,
            )
        elif shot_type == SHOT_CURVE:
            profile.update(
                power_bonus=1.02,
                curve_scale=1.55,
                lift_bonus=1.08,
                accuracy=accuracy + 0.08,
                post_chance=0.12,
                top_bins_chance=0.32,
                curve_blend=0.0,
            )
        elif shot_type == SHOT_KNUCKLE:
            profile.update(
                power_bonus=1.22,
                curve_scale=0.0,
                lift_bonus=1.12,
                accuracy=accuracy - 0.16,
                post_chance=0.05,
                top_bins_chance=0.14,
                curve_blend=0.30,
                knuckle_strength=3.8,
            )
        elif shot_type == SHOT_VOLLEY:
            profile.update(
                power_bonus=1.18,
                curve_scale=0.76,
                lift_bonus=1.28,
                accuracy=accuracy - 0.05,
                post_chance=0.09,
                top_bins_chance=0.24,
                curve_blend=0.08,
            )

        if self.name == "Cristiano Ronaldo":
            if shot_type in (SHOT_POWER, SHOT_KNUCKLE, SHOT_VOLLEY):
                profile["power_bonus"] *= 1.12
                profile["lift_bonus"] *= 1.04
            profile["curve_scale"] *= 1.08 if shot_type != SHOT_KNUCKLE else 0.0
            profile["accuracy"] -= 0.04 if shot_type in (SHOT_POWER, SHOT_KNUCKLE) else 0.02
            profile["knuckle_strength"] *= 1.28
        elif self.name == "Lionel Messi":
            if shot_type in (SHOT_NORMAL, SHOT_CURVE, SHOT_VOLLEY):
                profile["curve_scale"] *= 1.30
                profile["accuracy"] += 0.08
                profile["top_bins_chance"] += 0.05
            if shot_type in (SHOT_POWER, SHOT_KNUCKLE):
                profile["power_bonus"] *= 0.96
                profile["accuracy"] += 0.03
            profile["knuckle_strength"] *= 0.70
        elif self.name == "Karim Benzema":
            profile["power_bonus"] *= 1.04
            profile["accuracy"] += 0.02
        elif self.name == "Luis Suarez":
            profile["power_bonus"] *= 1.06
            profile["curve_scale"] *= 1.08
            profile["accuracy"] += 0.01

        profile["accuracy"] = clamp(profile["accuracy"], 0.55, 0.97)
        return profile

    def get_shot_target_profile(self, shot_type, target_goal_center, charge, shot_profile):
        open_side = self.get_open_goal_side()
        target_x = None
        curve_force = 0.0
        lift_boost = 0.0

        if shot_type == SHOT_NORMAL:
            curve_force, curve_target_x = self.get_curve_profile(target_goal_center, charge)
            if curve_force > 0.01:
                target_x = curve_target_x
            else:
                straight_curve = self.get_straight_curve_profile(target_goal_center, charge)
                if straight_curve is not None:
                    target_x, curve_force = straight_curve
            keeper_curve = self.get_goalkeeper_curve_profile(target_goal_center, charge)
            if keeper_curve is not None:
                keeper_target, keeper_force = keeper_curve
                if keeper_force > curve_force:
                    target_x = keeper_target
                    curve_force = keeper_force
        elif shot_type == SHOT_POWER:
            target_x = open_side * random.uniform(GOAL_WIDTH * 0.04, GOAL_WIDTH * 0.16)
            curve_force = SHOT_STRAIGHT_CURVE_FORCE * 0.20
        elif shot_type == SHOT_CURVE:
            target_x = open_side * random.uniform(GOAL_WIDTH * 0.22, GOAL_WIDTH * 0.40)
            curve_force = SHOT_KEEPER_CURVE_FORCE * 1.04
            keeper_curve = self.get_goalkeeper_curve_profile(target_goal_center, charge)
            if keeper_curve is not None:
                keeper_target, keeper_force = keeper_curve
                target_x = keeper_target
                curve_force = max(curve_force, keeper_force * 1.08)
        elif shot_type == SHOT_KNUCKLE:
            target_x = random.uniform(-GOAL_WIDTH * 0.12, GOAL_WIDTH * 0.12)
            lift_boost = 18.0
        elif shot_type == SHOT_VOLLEY:
            target_x = open_side * random.uniform(GOAL_WIDTH * 0.14, GOAL_WIDTH * 0.30)
            curve_force = SHOT_CURVE_FORCE * 0.62
            lift_boost = 46.0

        if target_x is None:
            target_x = open_side * random.uniform(GOAL_WIDTH * 0.06, GOAL_WIDTH * 0.20)

        aim_roll = random.random()
        if aim_roll < shot_profile["post_chance"]:
            target_x = open_side * (GOAL_WIDTH * 0.5 - BALL_RADIUS * 1.1)
            lift_boost += 10.0
        elif aim_roll < shot_profile["post_chance"] + shot_profile["top_bins_chance"]:
            target_x += open_side * random.uniform(GOAL_WIDTH * 0.05, GOAL_WIDTH * 0.12)
            lift_boost += 52.0 if shot_type != SHOT_POWER else 36.0

        if random.random() > shot_profile["accuracy"]:
            target_x += random.choice((-1.0, 1.0)) * random.uniform(GOAL_WIDTH * 0.10, GOAL_WIDTH * 0.24)

        spread = lerp(50.0, 6.0, shot_profile["accuracy"])
        target_x += random.uniform(-spread, spread) * (1.08 - shot_profile["accuracy"])
        curve_force *= shot_profile["curve_scale"]
        return clamp(target_x, -PITCH_WIDTH * 0.42, PITCH_WIDTH * 0.42), max(0.0, curve_force), lift_boost

    def get_shot_curve(self, shot_direction, target_goal_center, target_x_override, curve_force, charge, curve_input=0.0):
        if target_x_override is None or curve_force <= 0.01:
            return 0.0, BALL_CURVE_DECAY

        curve_target = V2(target_x_override, target_goal_center.y)
        to_target = curve_target - self.pos
        if to_target.length_squared() <= 0.01:
            return 0.0, BALL_CURVE_DECAY

        if abs(curve_input) > 0.01:
            curve_sign = 1.0 if curve_input > 0.0 else -1.0
        else:
            cross_value = shot_direction.cross(to_target.normalize())
            if abs(cross_value) > 0.01:
                curve_sign = 1.0 if cross_value > 0.0 else -1.0
            else:
                curve_sign = 1.0 if target_x_override < self.pos.x else -1.0

        charge_curve_scale, _ = self.get_shot_curve_profile(charge)
        input_curve_scale = 0.55 + abs(curve_input) * 0.90
        force_scale = clamp(curve_force / SHOT_KEEPER_CURVE_FORCE, 0.08, 1.36) * input_curve_scale
        base_curve = self.stats.curve_power * force_scale * charge_curve_scale * BALL_CURVE_ACCEL
        random_curve = random.uniform(-0.10, 0.10) * (1.10 - self.stats.curve_control)
        curve_amount = clamp(max(0.0, base_curve + random_curve) * curve_sign, -42.0, 42.0)
        curve_decay = clamp(lerp(self.stats.curve_decay - 0.010, self.stats.curve_decay, charge), 0.95, 0.992)
        return curve_amount, curve_decay

    def attempt_shot(self, ball, charge, requested_type=SHOT_NORMAL):
        if self.kick_cooldown > 0:
            return False
        if ball.owner is not self and not self.can_control_ball(ball):
            return False

        power_ratio = clamp(charge, 0.18, 1.0)
        target_goal = self.game.get_goal_target(self.attack_direction)
        shot_type = self.resolve_shot_type(requested_type, charge, ball)
        shot_profile = self.get_player_shot_profile(shot_type)
        target_x_override, curve_force, lift_boost = self.get_shot_target_profile(shot_type, target_goal, charge, shot_profile)
        manual_target_x, manual_lift_boost, manual_curve_input = self.get_manual_shot_input(shot_type, power_ratio)
        if abs(manual_target_x) > 0.001:
            target_x_override = manual_target_x
        lift_boost += manual_lift_boost
        if abs(manual_curve_input) > 0.01 and shot_type != SHOT_KNUCKLE:
            curve_force = max(curve_force, abs(manual_curve_input) * SHOT_INPUT_CURVE_FORCE * shot_profile["curve_scale"])

        curve_blend = shot_profile["curve_blend"] if target_x_override is not None and curve_force > 0.01 else 1.0
        direction = self.get_shot_direction(charge, target_goal, target_x_override=target_x_override, curve_blend=curve_blend)
        curve_amount, curve_decay = self.get_shot_curve(
            direction,
            target_goal,
            target_x_override,
            curve_force,
            charge,
            curve_input=manual_curve_input,
        )
        if shot_type == SHOT_KNUCKLE:
            curve_amount = 0.0

        shot_power_scale, shot_lift_scale = self.get_shot_power_profile(charge)
        power = lerp(420.0, self.stats.shot_power * shot_power_scale * shot_profile["power_bonus"], charge)
        lift = 10.0 + self.stats.shot_lift * shot_lift_scale * shot_profile["lift_bonus"] * (power_ratio ** 1.45) + lift_boost
        ball.kick(
            direction,
            power,
            lift,
            self,
            curve_amount=curve_amount,
            curve_decay=curve_decay,
            curve_target_x=target_x_override,
            curve_timer=SHOT_CURVE_DURATION,
            knuckle_strength=shot_profile["knuckle_strength"],
            shot_type=shot_type,
        )
        self.kick_cooldown = 0.32
        self.shoot_flash = 0.16
        self.game.message(SHOT_LABELS[shot_type], 0.40)
        self.game.play_sound("kick")
        return True


class Goalkeeper3D:
    def __init__(self, game, name, team_name, defend_z, attack_direction, colors):
        self.game = game
        self.name = name
        self.team_name = team_name
        self.defend_z = defend_z
        self.attack_direction = attack_direction
        self.colors = colors
        self.pos = V2(0, defend_z)
        self.vel = V2()
        self.height = GOALKEEPER_HEIGHT
        self.save_cooldown = 0.0

    def reset(self):
        self.pos.update(0, self.defend_z)
        self.vel.update(0, 0)
        self.save_cooldown = 0.0

    def update(self, dt):
        if self.save_cooldown > 0:
            self.save_cooldown = max(0.0, self.save_cooldown - dt)

        ball = self.game.ball
        target_x = clamp(ball.pos.x * 0.78, -GOAL_WIDTH / 2 + 28, GOAL_WIDTH / 2 - 28)
        target_z = self.defend_z
        if ball.owner is not None and ball.owner.team_name != self.team_name and abs(ball.pos.z - self.defend_z) < 180:
            target_z += 18 if self.attack_direction > 0 else -18

        target = V2(target_x, target_z)
        delta = target - self.pos
        if delta.length_squared() > 1.0:
            desired = delta.normalize() * GOALKEEPER_MOVE_SPEED
            self.vel = self.vel.lerp(desired, min(1.0, 6.8 * dt))
            self.pos += self.vel * dt
        else:
            self.vel *= 0.6

        self.pos.x = clamp(self.pos.x, -GOAL_WIDTH / 2 + 24, GOAL_WIDTH / 2 - 24)
        self.try_save()

    def try_save(self):
        if self.game.play_state != "PLAYING" or self.save_cooldown > 0:
            return

        ball = self.game.ball
        if ball.owner is not None:
            owner = ball.owner
            if owner.team_name == self.team_name:
                return
            if self.pos.distance_to(owner.pos) <= GOALKEEPER_OWNER_SAVE_REACH:
                clear_dir = V2(random.uniform(-0.35, 0.35), -1 if self.attack_direction < 0 else 1)
                ball.kick(clear_dir.normalize(), 360.0, 145.0, self)
                ball.pickup_cooldown = 0.24
                self.save_cooldown = 0.42
                self.game.message("SAVE!", 0.45)
                self.game.play_sound("kick")
            return

        incoming = ball.vel.z < 0 if self.attack_direction < 0 else ball.vel.z > 0
        if not incoming or ball.pos.y > GOAL_HEIGHT * 0.80:
            return

        if self.pos.distance_to(V2(ball.pos.x, ball.pos.z)) <= GOALKEEPER_SHOT_SAVE_REACH:
            clear_dir = V2(random.uniform(-0.32, 0.32), -1 if self.attack_direction < 0 else 1).normalize()
            clear_power = max(280.0, ball.horizontal_speed() * 0.70)
            ball.kick(clear_dir, clear_power, 110.0, self)
            ball.pickup_cooldown = 0.30
            self.save_cooldown = 0.42
            self.game.message("SAVE!", 0.45)
            self.game.play_sound("kick")


class Ball3D:
    def __init__(self, game):
        self.game = game
        self.pos = V3(0.0, 0.0, PITCH_LENGTH * 0.5)
        self.previous_pos = V3(self.pos)
        self.vel = V3()
        self.owner = None
        self.last_touch_team = None
        self.pickup_cooldown = 0.0
        self.pass_target = None
        self.pass_timer = 0.0
        self.curve = 0.0
        self.curve_decay = BALL_CURVE_DECAY
        self.curve_target_x = 0.0
        self.curve_timer = 0.0
        self.knuckle_strength = 0.0
        self.knuckle_decay = KNUCKLE_DECAY
        self.last_shot_type = SHOT_NORMAL
        self.trail = []

    def reset(self, position, owner=None):
        self.pos.update(position)
        self.previous_pos.update(position)
        self.vel.update(0, 0, 0)
        self.owner = owner
        self.last_touch_team = owner.team_name if owner else None
        self.pickup_cooldown = 0.22
        self.pass_target = None
        self.pass_timer = 0.0
        self.curve = 0.0
        self.curve_decay = BALL_CURVE_DECAY
        self.curve_target_x = 0.0
        self.curve_timer = 0.0
        self.knuckle_strength = 0.0
        self.knuckle_decay = KNUCKLE_DECAY
        self.last_shot_type = SHOT_NORMAL
        self.trail.clear()

        if owner is not None:
            owner.possession_buffer = POSSESSION_PROTECTION_TIME

    def horizontal_speed(self):
        return math.hypot(self.vel.x, self.vel.z)

    def set_owner(self, player, protection=POSSESSION_PROTECTION_TIME):
        self.owner = player
        self.last_touch_team = player.team_name
        self.pickup_cooldown = 0.14
        self.pass_target = None
        self.pass_timer = 0.0
        self.curve = 0.0
        self.curve_decay = BALL_CURVE_DECAY
        self.curve_target_x = 0.0
        self.curve_timer = 0.0
        self.knuckle_strength = 0.0
        self.knuckle_decay = KNUCKLE_DECAY
        self.last_shot_type = SHOT_NORMAL
        self.vel.update(0.0, 0.0, 0.0)
        player.possession_buffer = protection

    def kick(
        self,
        direction,
        power,
        lift,
        kicker,
        intended_receiver=None,
        pass_timer=0.0,
        curve_amount=0.0,
        curve_decay=BALL_CURVE_DECAY,
        curve_target_x=0.0,
        curve_timer=0.0,
        knuckle_strength=0.0,
        shot_type=SHOT_NORMAL,
    ):
        kick_direction = direction.normalize() if direction.length_squared() > 0.01 else V2(0, 1)
        origin_ground = V2(kicker.pos)
        contact_offset = kick_direction * 16.0
        self.pos.x = origin_ground.x + contact_offset.x
        self.pos.z = origin_ground.y + contact_offset.y
        self.pos.y = 10.0
        self.previous_pos.update(self.pos)
        self.owner = None
        self.last_touch_team = kicker.team_name
        self.pickup_cooldown = 0.18
        self.pass_target = intended_receiver
        self.pass_timer = pass_timer
        self.curve = curve_amount
        self.curve_decay = curve_decay
        self.curve_target_x = curve_target_x
        self.curve_timer = curve_timer
        self.knuckle_strength = knuckle_strength
        self.knuckle_decay = KNUCKLE_DECAY
        self.last_shot_type = shot_type
        self.vel.x = kick_direction.x * power
        self.vel.y = lift
        self.vel.z = kick_direction.y * power

    def attach_to_owner(self):
        owner = self.owner
        if owner is None:
            return

        forward = owner.facing.normalize() if owner.facing.length_squared() > 0.01 else V2(0, owner.attack_direction)
        offset = 22.0 + owner.stats.dribble * 7.0
        target = owner.pos + forward * offset
        lerp_speed = min(1.0, (12.0 + owner.stats.ball_control * 10.0) * self.game.dt)
        self.pos.x = lerp(self.pos.x, target.x, lerp_speed)
        self.pos.z = lerp(self.pos.z, target.y, lerp_speed)
        self.pos.y = lerp(self.pos.y, 3.0, min(1.0, 15.0 * self.game.dt))
        self.vel.x = owner.vel.x * 0.24
        self.vel.z = owner.vel.y * 0.24
        self.vel.y = 0.0

    def handle_side_collisions(self):
        side_limit = PITCH_WIDTH / 2 - BALL_RADIUS
        if self.pos.x < -side_limit:
            self.pos.x = -side_limit
            self.vel.x = abs(self.vel.x) * 0.52
        elif self.pos.x > side_limit:
            self.pos.x = side_limit
            self.vel.x = -abs(self.vel.x) * 0.52

    def handle_end_collisions(self):
        in_goal_mouth = abs(self.pos.x) <= GOAL_WIDTH / 2 - BALL_RADIUS and self.pos.y <= GOAL_HEIGHT
        if self.pos.z < 0.0:
            if in_goal_mouth:
                if self.pos.z < -GOAL_DEPTH + BALL_RADIUS:
                    self.pos.z = -GOAL_DEPTH + BALL_RADIUS
                    self.vel.z = abs(self.vel.z) * 0.34
            else:
                self.pos.z = 0.0
                self.vel.z = abs(self.vel.z) * 0.50
        elif self.pos.z > PITCH_LENGTH:
            if in_goal_mouth:
                if self.pos.z > PITCH_LENGTH + GOAL_DEPTH - BALL_RADIUS:
                    self.pos.z = PITCH_LENGTH + GOAL_DEPTH - BALL_RADIUS
                    self.vel.z = -abs(self.vel.z) * 0.34
            else:
                self.pos.z = PITCH_LENGTH
                self.vel.z = -abs(self.vel.z) * 0.50

    def try_pickup(self):
        if self.pickup_cooldown > 0 or self.owner is not None or self.game.play_state != "PLAYING":
            return

        candidates = []
        for player in self.game.outfield_players:
            if player.can_control_ball(self):
                distance = player.pos.distance_to(V2(self.pos.x, self.pos.z))
                score = player.stats.ball_control * 120.0 - distance * 1.9 - self.horizontal_speed() * 0.12
                if self.pass_target is player and self.pass_timer > 0:
                    score += PASS_RECEIVER_BONUS
                elif self.pass_target is not None and player.team_name != self.last_touch_team and self.pass_timer > 0:
                    score -= INTERCEPT_PENALTY
                candidates.append((score, player))

        if candidates:
            _, best_player = max(candidates, key=lambda item: item[0])
            self.set_owner(best_player)
            if best_player.team_name == REAL_MADRID:
                self.game.controlled_player = best_player

    def update(self, dt):
        self.previous_pos.update(self.pos)
        if self.pickup_cooldown > 0:
            self.pickup_cooldown = max(0.0, self.pickup_cooldown - dt)
        if self.pass_timer > 0:
            self.pass_timer = max(0.0, self.pass_timer - dt)
            if self.pass_timer == 0:
                self.pass_target = None
        if self.curve_timer > 0:
            self.curve_timer = max(0.0, self.curve_timer - dt)

        if self.owner is not None:
            self.attach_to_owner()
        else:
            horizontal_velocity = V2(self.vel.x, self.vel.z)
            # Bend the ball on the x/z plane every frame using a perpendicular force.
            if abs(self.curve) > 0.001 and horizontal_velocity.length_squared() > 0.5:
                perpendicular = V2(-horizontal_velocity.y, horizontal_velocity.x)
                if perpendicular.length_squared() > 0.0001:
                    perpendicular.scale_to_length(1.0)
                    horizontal_velocity += perpendicular * (self.curve * dt * FPS)
                    self.vel.x = horizontal_velocity.x
                    self.vel.z = horizontal_velocity.y
                self.curve *= self.curve_decay ** (dt * FPS)
                if abs(self.curve) < 0.01:
                    self.curve = 0.0
            if self.knuckle_strength > 0.01 and self.pos.y > VOLLEY_TRIGGER_HEIGHT:
                self.vel.x += random.uniform(-self.knuckle_strength, self.knuckle_strength)
                self.vel.z += random.uniform(-self.knuckle_strength, self.knuckle_strength)
                self.knuckle_strength *= self.knuckle_decay ** (dt * FPS)
                if self.knuckle_strength < 0.05:
                    self.knuckle_strength = 0.0
            self.vel.y -= BALL_GRAVITY * dt
            drag = BALL_AIR_DRAG if self.pos.y > 1.0 else BALL_GROUND_DRAG
            self.vel.x *= max(0.0, 1.0 - drag * dt)
            self.vel.z *= max(0.0, 1.0 - drag * dt)
            self.pos += self.vel * dt

            if self.pos.y <= 0.0:
                self.pos.y = 0.0
                if abs(self.vel.y) > BALL_MIN_BOUNCE_SPEED:
                    self.vel.y *= -BALL_BOUNCE
                    self.vel.x *= 0.95
                    self.vel.z *= 0.95
                else:
                    self.vel.y = 0.0

            if abs(self.vel.x) < 6.0:
                self.vel.x = 0.0
            if abs(self.vel.z) < 6.0:
                self.vel.z = 0.0

            self.handle_side_collisions()
            self.handle_end_collisions()
            self.try_pickup()

        self.trail.insert(0, V3(self.pos))
        del self.trail[BALL_TRAIL_LENGTH:]


class AIController:
    IDLE = "IDLE"
    ATTACK = "ATTACK"
    DEFEND = "DEFEND"

    def __init__(self, player):
        self.player = player
        self.state = self.IDLE
        self.timer = 0.0
        self.move_intent = V2()
        self.sprint = False
        self.charge = 0.0
        self.shot_type = SHOT_NORMAL

    def update(self, dt, game):
        self.timer += dt
        if self.timer < AI_UPDATE_INTERVAL:
            return

        self.timer = 0.0
        self.charge = 0.0
        self.shot_type = SHOT_NORMAL
        player = self.player
        ball = game.ball
        ball_ground = V2(ball.pos.x, ball.pos.z)
        own_goal = game.get_team_goal(player.team_name)
        enemy_goal = game.get_attack_goal(player.team_name)
        teammates = game.get_teammates(player)
        opponents = game.get_opponents(player)

        if game.play_state != "PLAYING":
            self.state = self.IDLE
            self.move_intent.update(0, 0)
            self.sprint = False
            return

        if ball.owner is player:
            self.state = self.ATTACK
            target = V2(clamp(player.spawn_pos.x * 0.55 - player.pos.x * 0.18, -90.0, 90.0), 0.0) + enemy_goal
            run_vector = target - player.pos
            self.move_intent = run_vector.normalize() if run_vector.length_squared() > 0.01 else V2(0, -1)
            self.sprint = run_vector.length_squared() > 14000.0
            goal_distance = player.pos.distance_to(enemy_goal)
            if ball.pos.y > VOLLEY_TRIGGER_HEIGHT:
                self.charge = 0.70
                self.shot_type = SHOT_VOLLEY
            if goal_distance < 295.0 and abs(player.pos.x) < GOAL_WIDTH * 0.74:
                if player.name == "Lionel Messi":
                    self.charge = 0.76
                    self.shot_type = SHOT_CURVE
                elif player.name == "Cristiano Ronaldo":
                    self.charge = 0.90 if goal_distance > 240.0 else 0.78
                    self.shot_type = SHOT_KNUCKLE if goal_distance > 315.0 else SHOT_POWER
                elif player.name == "Karim Benzema":
                    self.charge = 0.80
                    self.shot_type = SHOT_POWER
                else:
                    self.charge = 0.74
                    self.shot_type = SHOT_CURVE if abs(player.pos.x) > GOAL_WIDTH * 0.18 else SHOT_NORMAL
        elif ball.owner is not None and ball.owner.team_name == player.team_name:
            self.state = self.ATTACK
            carrier = ball.owner
            support_target = carrier.pos + V2(
                clamp(player.spawn_pos.x - carrier.pos.x * 0.20, -150.0, 150.0),
                -player.attack_direction * 88.0,
            )
            support_target.x = clamp(support_target.x, -PITCH_WIDTH / 2 + 36, PITCH_WIDTH / 2 - 36)
            support_target.y = clamp(support_target.y, 48.0, PITCH_LENGTH - 48.0)
            support_move = support_target - player.pos
            self.move_intent = support_move.normalize() if support_move.length_squared() > 0.01 else V2()
            self.sprint = support_move.length_squared() > 9800.0
        elif ball.owner is None:
            closest_for_team = game.get_closest_player_to_ball(player.team_name)
            if closest_for_team is player:
                self.state = self.ATTACK
                chase = ball_ground - player.pos
                self.move_intent = chase.normalize() if chase.length_squared() > 0.01 else V2()
                self.sprint = chase.length_squared() > 9000.0
            else:
                self.state = self.DEFEND
                hold_target = V2(player.spawn_pos.x * 0.92, clamp(ball_ground.y - player.attack_direction * 92.0, 72.0, PITCH_LENGTH - 72.0))
                hold_move = hold_target - player.pos
                self.move_intent = hold_move.normalize() if hold_move.length_squared() > 0.01 else V2()
                self.sprint = hold_move.length_squared() > 6800.0
        else:
            self.state = self.DEFEND
            carrier = ball.owner
            primary_chaser = game.get_closest_player_to_ball(player.team_name)
            if primary_chaser is player:
                intercept = ball_ground.lerp(own_goal, 0.20)
            else:
                mark_target = min(opponents, key=lambda opponent: opponent.pos.distance_to(player.pos)) if opponents else carrier
                intercept = mark_target.pos.lerp(own_goal, 0.42)
            intercept.x = clamp(intercept.x, -PITCH_WIDTH / 2 + 30, PITCH_WIDTH / 2 - 30)
            intercept.y = clamp(intercept.y, 38.0, PITCH_LENGTH - 38.0)
            move = intercept - player.pos
            self.move_intent = move.normalize() if move.length_squared() > 0.01 else V2()
            self.sprint = move.length_squared() > 6200.0


class Game3D:
    def __init__(self):
        pygame.display.set_caption("Football 3D - Real Madrid vs Barcelona")
        self.screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
        self.clock = pygame.time.Clock()
        self.running = True
        self.dt = 0.0

        self.ui_font = pygame.font.Font(None, 34)
        self.small_font = pygame.font.Font(None, 24)
        self.big_font = pygame.font.Font(None, 80)
        self.text_cache = {}
        self.shadow_cache = {}
        self.trail_cache = {}
        self.ball_surface_cache = {}
        self.sounds = self.build_sounds()
        self.background_surface = self.build_background_surface()
        self.ui_panel_surface = pygame.Surface((SCREEN_WIDTH, 86), pygame.SRCALPHA)
        self.ui_panel_surface.fill(HUD_PANEL)
        self.help_surface = self.get_cached_text(
            self.small_font,
            "WASD move  Shift sprint  Arrows aim shot  Tap/Hold Space normal/power  Q curve  E knuckle  R pass",
            (224, 224, 224),
        )

        self.human = Player3D(
            self,
            "Cristiano Ronaldo",
            REAL_MADRID,
            RONALDO_STATS,
            attack_direction=1,
            start_pos=(-125.0, 220.0),
            colors=((244, 244, 244), (176, 156, 56)),
            label="CR7",
        )
        self.home_support = Player3D(
            self,
            "Karim Benzema",
            REAL_MADRID,
            BENZEMA_STATS,
            attack_direction=1,
            start_pos=(135.0, 315.0),
            colors=((244, 244, 244), (116, 116, 116)),
            label="KB9",
        )
        self.ai_player = Player3D(
            self,
            "Lionel Messi",
            BARCELONA,
            MESSI_STATS,
            attack_direction=-1,
            start_pos=(125.0, PITCH_LENGTH - 220.0),
            colors=((33, 75, 165), (170, 35, 49)),
            label="LM10",
        )
        self.away_support = Player3D(
            self,
            "Luis Suarez",
            BARCELONA,
            SUAREZ_STATS,
            attack_direction=-1,
            start_pos=(-135.0, PITCH_LENGTH - 315.0),
            colors=((33, 75, 165), (102, 183, 221)),
            label="LS9",
        )
        self.home_outfield = [self.human, self.home_support]
        self.away_outfield = [self.ai_player, self.away_support]
        self.outfield_players = self.home_outfield + self.away_outfield
        self.controlled_player = self.human
        self.home_keeper = Goalkeeper3D(self, "Courtois", REAL_MADRID, 42.0, -1, ((123, 196, 255), (44, 95, 168)))
        self.away_keeper = Goalkeeper3D(
            self,
            "ter Stegen",
            BARCELONA,
            PITCH_LENGTH - 42.0,
            1,
            ((255, 188, 112), (152, 76, 28)),
        )
        self.ball = Ball3D(self)
        self.ai_controllers = {player: AIController(player) for player in self.outfield_players}

        self.camera_x = 0.0
        self.camera_z = -90.0
        self.scores = {REAL_MADRID: 0, BARCELONA: 0}
        self.match_time = MATCH_TIME_SECONDS
        self.play_state = "STOPPED"
        self.stop_timer = 1.2
        self.pending_kickoff = REAL_MADRID
        self.overlay_text = "KICK OFF"
        self.overlay_timer = 1.2
        self.action_text = ""
        self.action_timer = 0.0
        self.shot_held = False
        self.shot_hold_key = None
        self.shot_request_type = SHOT_NORMAL
        self.shot_hold_time = 0.0
        self.charge_value = 0.0
        self.shot_aim_x = 0.0
        self.shot_lift_input = 0.0
        self.shot_curve_input = 0.0
        self.steal_cooldown = 0.0

        self.reset_positions(REAL_MADRID)

    def get_cached_text(self, font, text, color):
        key = (id(font), text, color)
        surface = self.text_cache.get(key)
        if surface is None:
            surface = font.render(text, True, color)
            self.text_cache[key] = surface
        return surface

    def build_background_surface(self):
        surface = pygame.Surface((SCREEN_WIDTH, SCREEN_HEIGHT))
        for row in range(SCREEN_HEIGHT):
            blend = row / SCREEN_HEIGHT
            color = (
                int(lerp(SKY_TOP[0], SKY_BOTTOM[0], blend)),
                int(lerp(SKY_TOP[1], SKY_BOTTOM[1], blend)),
                int(lerp(SKY_TOP[2], SKY_BOTTOM[2], blend)),
            )
            pygame.draw.line(surface, color, (0, row), (SCREEN_WIDTH, row))
        pygame.draw.rect(surface, (74, 78, 93), (0, HORIZON_Y - 18, SCREEN_WIDTH, 22))
        pygame.draw.rect(surface, (104, 108, 124), (0, HORIZON_Y + 2, SCREEN_WIDTH, 18))
        return surface

    def get_shadow_surface(self, width, height, alpha):
        key = (width, height, alpha)
        surface = self.shadow_cache.get(key)
        if surface is None:
            surface = pygame.Surface((width * 2, height * 2), pygame.SRCALPHA)
            pygame.draw.ellipse(surface, (0, 0, 0, alpha), surface.get_rect())
            self.shadow_cache[key] = surface
        return surface

    def get_trail_surface(self, radius, alpha):
        key = (radius, alpha)
        surface = self.trail_cache.get(key)
        if surface is None:
            surface = pygame.Surface((radius * 2 + 2, radius * 2 + 2), pygame.SRCALPHA)
            pygame.draw.circle(surface, (255, 255, 255, alpha), (radius + 1, radius + 1), radius)
            self.trail_cache[key] = surface
        return surface

    def get_ball_surface(self, radius):
        surface = self.ball_surface_cache.get(radius)
        if surface is None:
            surface = pygame.Surface((radius * 2 + 4, radius * 2 + 4), pygame.SRCALPHA)
            center = (radius + 2, radius + 2)
            pygame.draw.circle(surface, (244, 244, 244), center, radius)
            pygame.draw.circle(surface, (44, 44, 44), center, radius, 2)
            self.ball_surface_cache[radius] = surface
        return surface

    def build_sounds(self):
        sounds = {}
        try:
            sounds["kick"] = create_tone(520, 85, volume=0.28)
            sounds["goal"] = create_tone(860, 180, volume=0.38)
        except pygame.error:
            sounds["kick"] = None
            sounds["goal"] = None
        return sounds

    def play_sound(self, name):
        sound = self.sounds.get(name)
        if sound is not None:
            sound.play()

    def message(self, text, duration):
        self.action_text = text
        self.action_timer = duration

    def get_goal_target(self, attack_direction):
        return V2(0.0, PITCH_LENGTH + GOAL_DEPTH * 0.20) if attack_direction > 0 else V2(0.0, -GOAL_DEPTH * 0.20)

    def get_target_goalkeeper(self, attack_direction):
        return self.away_keeper if attack_direction > 0 else self.home_keeper

    def get_team_players(self, team_name):
        return self.home_outfield if team_name == REAL_MADRID else self.away_outfield

    def get_teammates(self, player):
        return [teammate for teammate in self.get_team_players(player.team_name) if teammate is not player]

    def get_opponents(self, player):
        return self.away_outfield if player.team_name == REAL_MADRID else self.home_outfield

    def get_team_goal(self, team_name):
        return V2(0.0, 0.0) if team_name == REAL_MADRID else V2(0.0, PITCH_LENGTH)

    def get_attack_goal(self, team_name):
        return V2(0.0, PITCH_LENGTH) if team_name == REAL_MADRID else V2(0.0, 0.0)

    def get_closest_player_to_ball(self, team_name):
        ball_ground = V2(self.ball.pos.x, self.ball.pos.z)
        return min(self.get_team_players(team_name), key=lambda player: player.pos.distance_to(ball_ground))

    def find_best_pass_target(self, passer):
        teammates = self.get_teammates(passer)
        if not teammates:
            return None

        opponents = self.get_opponents(passer)
        best_target = None
        best_score = -10_000.0
        for teammate in teammates:
            pass_vector = teammate.pos - passer.pos
            distance = pass_vector.length()
            if distance < 52.0 or distance > 320.0:
                continue

            forward_progress = passer.attack_direction * (teammate.pos.y - passer.pos.y)
            receiver_space = min(opponent.pos.distance_to(teammate.pos) for opponent in opponents) if opponents else 90.0
            lane_penalty = 0.0
            for opponent in opponents:
                lane_penalty += max(0.0, 48.0 - point_segment_distance(opponent.pos, passer.pos, teammate.pos))

            score = (
                forward_progress * 1.35
                + receiver_space * 0.82
                - abs(distance - 150.0) * 0.34
                - lane_penalty * 1.7
            )
            if score > best_score:
                best_score = score
                best_target = teammate
        return best_target

    def attempt_pass(self, passer):
        if self.ball.owner is not passer and not passer.can_control_ball(self.ball):
            return False

        target = self.find_best_pass_target(passer)
        if target is None:
            return False

        lead_target = target.pos + target.vel * PASS_LEAD_FACTOR + V2(0.0, passer.attack_direction * 20.0)
        pass_vector = lead_target - passer.pos
        if pass_vector.length_squared() <= 0.01:
            return False

        distance = pass_vector.length()
        power = clamp(distance * 3.25 + 120.0, PASS_MIN_POWER, PASS_MAX_POWER)
        lift = clamp(34.0 + distance * 0.12, 32.0, 74.0)
        self.ball.kick(pass_vector.normalize(), power, lift, passer, intended_receiver=target, pass_timer=0.9)
        passer.kick_cooldown = max(passer.kick_cooldown, 0.22)
        passer.shoot_flash = 0.10
        self.controlled_player = target
        self.message(f"PASS TO {target.label}", 0.45)
        self.play_sound("kick")
        return True

    def reset_positions(self, kickoff_team):
        self.pending_kickoff = kickoff_team
        self.human.reset((-125.0, 220.0))
        self.home_support.reset((135.0, 315.0))
        self.ai_player.reset((125.0, PITCH_LENGTH - 220.0))
        self.away_support.reset((-135.0, PITCH_LENGTH - 315.0))
        self.home_keeper.reset()
        self.away_keeper.reset()
        owner = self.human if kickoff_team == REAL_MADRID else self.ai_player
        self.ball.reset(V3(0.0, 0.0, PITCH_LENGTH * 0.5), owner=owner)
        self.controlled_player = self.human
        self.shot_held = False
        self.shot_hold_key = None
        self.shot_request_type = SHOT_NORMAL
        self.shot_hold_time = 0.0
        self.charge_value = 0.0
        self.shot_aim_x = 0.0
        self.shot_lift_input = 0.0
        self.shot_curve_input = 0.0
        self.steal_cooldown = 0.0

    def begin_stoppage(self, text, duration, kickoff_team):
        self.play_state = "STOPPED"
        self.stop_timer = duration
        self.overlay_text = text
        self.overlay_timer = duration
        self.pending_kickoff = kickoff_team
        for player in self.outfield_players:
            player.vel *= 0.0
        self.ball.vel *= 0.0
        self.ball.owner = None
        self.shot_held = False
        self.shot_hold_key = None
        self.shot_request_type = SHOT_NORMAL
        self.shot_hold_time = 0.0
        self.shot_aim_x = 0.0
        self.shot_lift_input = 0.0
        self.shot_curve_input = 0.0

    def handle_goal(self, scoring_team):
        self.scores[scoring_team] += 1
        self.play_sound("goal")
        kickoff_team = BARCELONA if scoring_team == REAL_MADRID else REAL_MADRID
        self.begin_stoppage("GOAL!", 1.8, kickoff_team)

    def crossed_goal_line(self, side):
        if side == "far":
            return (
                self.ball.previous_pos.z <= PITCH_LENGTH
                and self.ball.pos.z >= PITCH_LENGTH
                and abs(self.ball.pos.x) <= GOAL_WIDTH / 2 - BALL_RADIUS
                and self.ball.pos.y <= GOAL_HEIGHT
            )
        return (
            self.ball.previous_pos.z >= 0.0
            and self.ball.pos.z <= 0.0
            and abs(self.ball.pos.x) <= GOAL_WIDTH / 2 - BALL_RADIUS
            and self.ball.pos.y <= GOAL_HEIGHT
        )

    def update_camera(self, dt):
        focus_x = self.controlled_player.pos.x * 0.55 + self.ball.pos.x * 0.45
        focus_z = self.controlled_player.pos.y * 0.50 + self.ball.pos.z * 0.50
        target_x = focus_x * 0.42
        target_z = clamp(focus_z - CAMERA_BACK_OFFSET, -110.0, PITCH_LENGTH * 0.44)
        self.camera_x = lerp(self.camera_x, target_x, min(1.0, CAMERA_X_DAMPING * dt))
        self.camera_z = lerp(self.camera_z, target_z, min(1.0, CAMERA_Z_DAMPING * dt))

    def project_point(self, x, y, z):
        relative_x = x - self.camera_x
        relative_z = z - self.camera_z
        if relative_z <= NEAR_PLANE:
            return None
        scale = PROJECT_SCALE / relative_z
        screen_x = SCREEN_WIDTH * 0.5 + relative_x * scale
        screen_y = HORIZON_Y + (CAMERA_HEIGHT - y) * scale
        return screen_x, screen_y, scale

    def project_ground(self, x, z):
        return self.project_point(x, 0.0, z)

    def draw_quad(self, points, color):
        projected = []
        for x, z in points:
            point = self.project_ground(x, z)
            if point is None:
                return
            projected.append((point[0], point[1]))
        pygame.draw.polygon(self.screen, color, projected)

    def draw_field_line(self, start, end, width=3):
        p1 = self.project_ground(start[0], start[1])
        p2 = self.project_ground(end[0], end[1])
        if p1 is None or p2 is None:
            return
        pygame.draw.line(self.screen, LINE_COLOR, (p1[0], p1[1]), (p2[0], p2[1]), width)

    def draw_arc_line(self, center, radius, start_angle, end_angle, steps=24):
        points = []
        for index in range(steps + 1):
            angle = lerp(start_angle, end_angle, index / steps)
            point = self.project_ground(center[0] + math.cos(angle) * radius, center[1] + math.sin(angle) * radius)
            if point is not None:
                points.append((point[0], point[1]))
        if len(points) >= 2:
            pygame.draw.lines(self.screen, LINE_COLOR, False, points, 3)

    def draw_background(self):
        self.screen.blit(self.background_surface, (0, 0))

    def draw_pitch(self):
        for stripe in range(10):
            z0 = PITCH_LENGTH * stripe / 10
            z1 = PITCH_LENGTH * (stripe + 1) / 10
            color = GRASS_A if stripe % 2 == 0 else GRASS_B
            self.draw_quad([(-PITCH_WIDTH / 2, z0), (PITCH_WIDTH / 2, z0), (PITCH_WIDTH / 2, z1), (-PITCH_WIDTH / 2, z1)], color)

        self.draw_field_line((-PITCH_WIDTH / 2, 0.0), (PITCH_WIDTH / 2, 0.0), 4)
        self.draw_field_line((-PITCH_WIDTH / 2, PITCH_LENGTH), (PITCH_WIDTH / 2, PITCH_LENGTH), 4)
        self.draw_field_line((-PITCH_WIDTH / 2, 0.0), (-PITCH_WIDTH / 2, PITCH_LENGTH), 4)
        self.draw_field_line((PITCH_WIDTH / 2, 0.0), (PITCH_WIDTH / 2, PITCH_LENGTH), 4)
        self.draw_field_line((-PITCH_WIDTH / 2, PITCH_LENGTH * 0.5), (PITCH_WIDTH / 2, PITCH_LENGTH * 0.5), 3)
        self.draw_arc_line((0.0, PITCH_LENGTH * 0.5), CENTER_CIRCLE_RADIUS, 0.0, math.tau, 36)

        for box_depth, box_width in ((PENALTY_BOX_DEPTH, PENALTY_BOX_WIDTH), (SIX_BOX_DEPTH, SIX_BOX_WIDTH)):
            half_width = box_width * 0.5
            self.draw_field_line((-half_width, 0.0), (-half_width, box_depth), 3)
            self.draw_field_line((half_width, 0.0), (half_width, box_depth), 3)
            self.draw_field_line((-half_width, box_depth), (half_width, box_depth), 3)
            far_z = PITCH_LENGTH - box_depth
            self.draw_field_line((-half_width, PITCH_LENGTH), (-half_width, far_z), 3)
            self.draw_field_line((half_width, PITCH_LENGTH), (half_width, far_z), 3)
            self.draw_field_line((-half_width, far_z), (half_width, far_z), 3)

        self.draw_arc_line((0.0, 132.0), 56.0, math.pi * 0.12, math.pi * 0.88, 20)
        self.draw_arc_line((0.0, PITCH_LENGTH - 132.0), 56.0, -math.pi * 0.88, -math.pi * 0.12, 20)

    def draw_goal(self, goal_z, attack_direction):
        back_z = goal_z + GOAL_DEPTH * attack_direction
        points = {
            "fl": self.project_point(-GOAL_WIDTH / 2, 0.0, goal_z),
            "fr": self.project_point(GOAL_WIDTH / 2, 0.0, goal_z),
            "tl": self.project_point(-GOAL_WIDTH / 2, GOAL_HEIGHT, goal_z),
            "tr": self.project_point(GOAL_WIDTH / 2, GOAL_HEIGHT, goal_z),
            "bfl": self.project_point(-GOAL_WIDTH / 2, 0.0, back_z),
            "bfr": self.project_point(GOAL_WIDTH / 2, 0.0, back_z),
            "btl": self.project_point(-GOAL_WIDTH / 2, GOAL_HEIGHT, back_z),
            "btr": self.project_point(GOAL_WIDTH / 2, GOAL_HEIGHT, back_z),
        }
        if any(point is None for point in points.values()):
            return

        segments = [("fl", "tl"), ("fr", "tr"), ("tl", "tr"), ("fl", "bfl"), ("fr", "bfr"), ("tl", "btl"), ("tr", "btr"), ("bfl", "btl"), ("bfr", "btr"), ("btl", "btr")]
        for start, end in segments:
            p1, p2 = points[start], points[end]
            pygame.draw.line(self.screen, GOAL_FRAME, (p1[0], p1[1]), (p2[0], p2[1]), 3)

    def draw_shadow(self, x, z, width, height):
        point = self.project_ground(x, z)
        if point is None:
            return
        shadow_w = max(8, int(width * point[2] * 0.80))
        shadow_h = max(4, int(height * point[2] * 0.22))
        shadow_surface = self.get_shadow_surface(shadow_w, shadow_h, SHADOW_COLOR[3])
        self.screen.blit(shadow_surface, shadow_surface.get_rect(center=(int(point[0]), int(point[1] + 3))))

    def draw_player(self, player):
        foot = self.project_point(player.pos.x, 0.0, player.pos.y)
        head = self.project_point(player.pos.x, player.height, player.pos.y)
        if foot is None or head is None:
            return
        height_px = foot[1] - head[1]
        if height_px <= 8:
            return

        width_px = max(12, int(height_px * 0.25))
        body_top = head[1] + height_px * 0.27
        torso_height = height_px * 0.45
        shorts_top = body_top + torso_height
        self.draw_shadow(player.pos.x, player.pos.y, 24.0, 8.0)

        jersey_color = player.colors[1] if player.shoot_flash > 0 else player.colors[0]
        accent_color = player.colors[1]
        body_rect = pygame.Rect(0, 0, width_px, int(torso_height))
        body_rect.midtop = (int(foot[0]), int(body_top))
        pygame.draw.rect(self.screen, jersey_color, body_rect, border_radius=8)

        stripe_rect = pygame.Rect(body_rect)
        stripe_rect.width = max(4, stripe_rect.width // 3)
        stripe_rect.centerx = body_rect.centerx
        pygame.draw.rect(self.screen, accent_color, stripe_rect, border_radius=4)

        shorts_rect = pygame.Rect(body_rect.left + 2, int(shorts_top), body_rect.width - 4, max(7, int(height_px * 0.15)))
        pygame.draw.rect(self.screen, (38, 38, 48), shorts_rect, border_radius=4)
        leg_w = max(4, width_px // 4)
        leg_h = max(9, int(height_px * 0.17))
        stride = math.sin(player.animation_time * 7.0) * max(1.0, width_px * 0.11)
        left_leg = pygame.Rect(int(foot[0] - width_px * 0.18 + stride), int(shorts_rect.bottom - 2), leg_w, leg_h)
        right_leg = pygame.Rect(int(foot[0] + width_px * 0.02 - stride), int(shorts_rect.bottom - 2), leg_w, leg_h)
        pygame.draw.rect(self.screen, (236, 236, 236), left_leg, border_radius=3)
        pygame.draw.rect(self.screen, (236, 236, 236), right_leg, border_radius=3)
        head_radius = max(5, int(height_px * 0.12))
        pygame.draw.circle(self.screen, (236, 210, 184), (int(head[0]), int(head[1] + head_radius)), head_radius)
        self.screen.blit(player.label_surface, player.label_surface.get_rect(center=(int(head[0]), int(head[1] - 14))))

    def draw_keeper(self, keeper):
        foot = self.project_point(keeper.pos.x, 0.0, keeper.pos.y)
        head = self.project_point(keeper.pos.x, keeper.height, keeper.pos.y)
        if foot is None or head is None:
            return
        height_px = foot[1] - head[1]
        if height_px <= 8:
            return

        width_px = max(13, int(height_px * 0.27))
        self.draw_shadow(keeper.pos.x, keeper.pos.y, 26.0, 8.0)
        body_rect = pygame.Rect(0, 0, width_px, int(height_px * 0.50))
        body_rect.midtop = (int(foot[0]), int(head[1] + height_px * 0.25))
        pygame.draw.rect(self.screen, keeper.colors[0], body_rect, border_radius=8)
        glove_rect = pygame.Rect(body_rect.left - 5, body_rect.top + 7, body_rect.width + 10, max(8, body_rect.height // 3))
        pygame.draw.rect(self.screen, keeper.colors[1], glove_rect, 2, border_radius=6)
        head_radius = max(5, int(height_px * 0.12))
        pygame.draw.circle(self.screen, (236, 210, 184), (int(head[0]), int(head[1] + head_radius)), head_radius)

    def draw_ball(self):
        render_radius = BALL_RADIUS * BALL_RENDER_SCALE
        for index, trail_pos in enumerate(self.ball.trail[1:], start=1):
            point = self.project_point(trail_pos.x, trail_pos.y, trail_pos.z)
            if point is None:
                continue
            alpha = max(0, 92 - index * 10)
            radius = max(2, int(render_radius * point[2] * (0.66 - index * 0.05)))
            if radius <= 0:
                continue
            trail_surface = self.get_trail_surface(radius, alpha)
            self.screen.blit(trail_surface, trail_surface.get_rect(center=(int(point[0]), int(point[1]))))

        shadow_point = self.project_ground(self.ball.pos.x, self.ball.pos.z)
        if shadow_point is not None:
            shadow_w = max(8, int(render_radius * shadow_point[2] * 1.45))
            shadow_h = max(4, int(render_radius * shadow_point[2] * 0.55))
            shadow_alpha = max(24, 105 - int(self.ball.pos.y * 0.50))
            shadow_surface = self.get_shadow_surface(shadow_w, shadow_h, shadow_alpha)
            self.screen.blit(shadow_surface, shadow_surface.get_rect(center=(int(shadow_point[0]), int(shadow_point[1] + 2))))

        point = self.project_point(self.ball.pos.x, self.ball.pos.y, self.ball.pos.z)
        if point is None:
            return
        radius = max(4, int(render_radius * point[2]))
        ball_surface = self.get_ball_surface(radius)
        self.screen.blit(ball_surface, ball_surface.get_rect(center=(int(point[0]), int(point[1]))))

    def draw_world(self):
        self.draw_background()
        self.draw_pitch()
        self.draw_goal(0.0, -1.0)
        self.draw_goal(PITCH_LENGTH, 1.0)

        renderables = [
            ("keeper", self.home_keeper.pos.y, self.home_keeper),
            ("keeper", self.away_keeper.pos.y, self.away_keeper),
            ("ball", self.ball.pos.z + self.ball.pos.y * 0.05, self.ball),
        ]
        renderables.extend(("player", player.pos.y, player) for player in self.outfield_players)
        renderables.sort(key=lambda item: item[1])
        for kind, _, obj in renderables:
            if kind == "player":
                self.draw_player(obj)
            elif kind == "keeper":
                self.draw_keeper(obj)
            else:
                self.draw_ball()

    def draw_ui(self):
        self.screen.blit(self.ui_panel_surface, (0, 0))

        timer_text = self.get_cached_text(self.ui_font, self.format_time(self.match_time), (255, 255, 255))
        score_text = self.get_cached_text(
            self.ui_font,
            f"{REAL_MADRID} {self.scores[REAL_MADRID]}  -  {self.scores[BARCELONA]} {BARCELONA}",
            (255, 255, 255),
        )
        self.screen.blit(score_text, score_text.get_rect(center=(SCREEN_WIDTH // 2, 30)))
        self.screen.blit(timer_text, timer_text.get_rect(center=(SCREEN_WIDTH // 2, 61)))
        self.screen.blit(self.help_surface, (18, 17))
        control_surface = self.get_cached_text(self.small_font, f"Control: {self.controlled_player.name}", (224, 224, 224))
        self.screen.blit(control_surface, (18, 42))

        if self.shot_held and self.play_state == "PLAYING":
            bar_rect = pygame.Rect(18, SCREEN_HEIGHT - 34, 240, 12)
            fill_rect = pygame.Rect(bar_rect)
            fill_rect.width = int(bar_rect.width * self.charge_value)
            pygame.draw.rect(self.screen, (30, 30, 36), bar_rect, border_radius=6)
            pygame.draw.rect(self.screen, (233, 205, 110), fill_rect, border_radius=6)
            pygame.draw.rect(self.screen, (250, 250, 250), bar_rect, 2, border_radius=6)
            shot_label = self.get_cached_text(self.small_font, self.get_shot_preview_label(), (244, 244, 244))
            self.screen.blit(shot_label, (18, SCREEN_HEIGHT - 56))

        if self.action_timer > 0:
            action_surface = self.get_cached_text(self.ui_font, self.action_text, (255, 236, 159))
            self.screen.blit(action_surface, action_surface.get_rect(center=(SCREEN_WIDTH // 2, 112)))

        if self.overlay_timer > 0 or self.play_state == "FINISHED":
            overlay_surface = self.get_cached_text(self.big_font, self.overlay_text, (255, 255, 255))
            self.screen.blit(overlay_surface, overlay_surface.get_rect(center=(SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2 - 30)))
            if self.play_state == "FINISHED":
                footer = self.get_cached_text(self.small_font, "Press Enter to restart", (242, 242, 242))
                self.screen.blit(footer, footer.get_rect(center=(SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2 + 34)))

    def format_time(self, seconds):
        total_seconds = max(0, int(seconds))
        return f"{total_seconds // 60:02d}:{total_seconds % 60:02d}"

    def get_shot_preview_label(self):
        if not self.shot_held:
            return ""
        if self.ball.owner is not self.controlled_player and self.ball.pos.y > VOLLEY_TRIGGER_HEIGHT:
            return SHOT_LABELS[SHOT_VOLLEY]
        if self.shot_request_type == SHOT_NORMAL:
            return SHOT_LABELS[SHOT_POWER] if self.charge_value >= POWER_SHOT_MIN_CHARGE else SHOT_LABELS[SHOT_NORMAL]
        return SHOT_LABELS[self.shot_request_type]

    def resolve_player_contact(self):
        for home_player in self.home_outfield:
            for away_player in self.away_outfield:
                delta = away_player.pos - home_player.pos
                distance = delta.length() or 1.0
                if delta.length_squared() == 0:
                    delta = V2(1, 0)
                if distance < 54.0:
                    push = (54.0 - distance) * 0.5
                    normal = delta / distance
                    home_player.pos -= normal * push
                    away_player.pos += normal * push

                if self.steal_cooldown > 0 or distance >= STEAL_DISTANCE:
                    continue

                ball_owner = self.ball.owner
                if ball_owner not in (home_player, away_player) or ball_owner.possession_buffer > 0:
                    continue

                holder = ball_owner
                thief = away_player if holder is home_player else home_player
                chance = clamp(
                    (STEAL_BASE_CHANCE + (thief.stats.ball_control - holder.stats.ball_control) * STEAL_SKILL_FACTOR)
                    * STEAL_DIFFICULTY_SCALE,
                    0.04,
                    0.18,
                )
                if random.random() < chance:
                    self.ball.set_owner(thief, protection=POSSESSION_PROTECTION_TIME * 0.90)
                    self.steal_cooldown = STEAL_COOLDOWN
                    if thief.team_name == REAL_MADRID:
                        self.controlled_player = thief
                    return

    def update_ai(self):
        for player, controller in self.ai_controllers.items():
            if player is self.controlled_player:
                continue
            controller.update(self.dt, self)
            player.intent = V2(controller.move_intent)
            player.sprint = controller.sprint
            if controller.charge > 0.0:
                player.attempt_shot(self.ball, controller.charge, controller.shot_type)
                controller.charge = 0.0

    def process_input(self):
        pressed = pygame.key.get_pressed()
        self.shot_aim_x = float(pressed[pygame.K_RIGHT]) - float(pressed[pygame.K_LEFT])
        self.shot_lift_input = float(pressed[pygame.K_UP]) - float(pressed[pygame.K_DOWN])
        self.shot_curve_input = float(pressed[pygame.K_d]) - float(pressed[pygame.K_a])

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                self.running = False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    self.running = False
                elif event.key in (pygame.K_SPACE, pygame.K_q, pygame.K_e) and self.play_state == "PLAYING" and not self.shot_held:
                    self.shot_held = True
                    self.shot_hold_key = event.key
                    if event.key == pygame.K_q:
                        self.shot_request_type = SHOT_CURVE
                    elif event.key == pygame.K_e:
                        self.shot_request_type = SHOT_KNUCKLE
                    else:
                        self.shot_request_type = SHOT_NORMAL
                    self.shot_hold_time = 0.0
                elif event.key == pygame.K_r and self.play_state == "PLAYING":
                    self.attempt_pass(self.controlled_player)
                elif event.key == pygame.K_RETURN and self.play_state == "FINISHED":
                    self.scores = {REAL_MADRID: 0, BARCELONA: 0}
                    self.match_time = MATCH_TIME_SECONDS
                    self.play_state = "STOPPED"
                    self.stop_timer = 1.2
                    self.overlay_text = "KICK OFF"
                    self.overlay_timer = 1.2
                    self.reset_positions(REAL_MADRID)
            elif event.type == pygame.KEYUP and self.shot_held and event.key == self.shot_hold_key:
                if self.play_state == "PLAYING":
                    self.controlled_player.attempt_shot(self.ball, self.charge_value, self.shot_request_type)
                self.shot_held = False
                self.shot_hold_key = None
                self.shot_request_type = SHOT_NORMAL
                self.shot_hold_time = 0.0

        if self.shot_held:
            move_x = float(pressed[pygame.K_d]) - float(pressed[pygame.K_a])
            move_z = float(pressed[pygame.K_w]) - float(pressed[pygame.K_s])
        else:
            move_x = float(pressed[pygame.K_d] or pressed[pygame.K_RIGHT]) - float(pressed[pygame.K_a] or pressed[pygame.K_LEFT])
            move_z = float(pressed[pygame.K_w] or pressed[pygame.K_UP]) - float(pressed[pygame.K_s] or pressed[pygame.K_DOWN])
        move = V2(move_x, move_z)
        if move.length_squared() > 1.0:
            move = move.normalize()
        self.controlled_player.intent = move
        self.controlled_player.sprint = bool(pressed[pygame.K_LSHIFT] or pressed[pygame.K_RSHIFT])

    def update_game(self):
        if self.action_timer > 0:
            self.action_timer = max(0.0, self.action_timer - self.dt)
            if self.action_timer == 0:
                self.action_text = ""
        if self.overlay_timer > 0 and self.play_state != "FINISHED":
            self.overlay_timer = max(0.0, self.overlay_timer - self.dt)
        if self.steal_cooldown > 0:
            self.steal_cooldown = max(0.0, self.steal_cooldown - self.dt)

        if self.shot_held and self.play_state == "PLAYING":
            self.shot_hold_time += self.dt
        self.charge_value = clamp(self.shot_hold_time / 0.72, 0.18, 1.0) if self.shot_held else 0.0

        if self.play_state == "PLAYING":
            self.match_time = max(0.0, self.match_time - self.dt)
            self.update_ai()
            for player in self.outfield_players:
                player.update(self.dt)
            self.resolve_player_contact()
            self.home_keeper.update(self.dt)
            self.away_keeper.update(self.dt)
            self.ball.update(self.dt)

            if self.crossed_goal_line("far"):
                self.handle_goal(REAL_MADRID)
            elif self.crossed_goal_line("near"):
                self.handle_goal(BARCELONA)

            if self.match_time <= 0:
                self.play_state = "FINISHED"
                if self.scores[REAL_MADRID] > self.scores[BARCELONA]:
                    self.overlay_text = "REAL MADRID WIN"
                elif self.scores[BARCELONA] > self.scores[REAL_MADRID]:
                    self.overlay_text = "BARCELONA WIN"
                else:
                    self.overlay_text = "FULL TIME DRAW"
                self.overlay_timer = 999.0
        elif self.play_state == "STOPPED":
            self.stop_timer -= self.dt
            for player in self.outfield_players:
                player.update(self.dt)
            self.home_keeper.update(self.dt)
            self.away_keeper.update(self.dt)
            if self.stop_timer <= 0:
                self.reset_positions(self.pending_kickoff)
                self.play_state = "PLAYING"
                self.overlay_timer = 0.0

        self.update_camera(self.dt)

    def render(self):
        self.draw_world()
        self.draw_ui()
        pygame.display.flip()

    def run(self):
        while self.running:
            self.dt = self.clock.tick(FPS) / 1000.0
            self.process_input()
            self.update_game()
            self.render()


def main():
    pygame.mixer.pre_init(22050, size=-16, channels=1, buffer=256)
    pygame.init()
    game = Game3D()
    game.run()
    pygame.quit()


def profile_main():
    import cProfile

    cProfile.run("main()")


if __name__ == "__main__":
    if "--profile" in sys.argv:
        profile_main()
    else:
        main()
