# GeneAlgorism
#유전적 알고리즘으로 경로 탐색
import numpy as np
import random as r

grid_map = [
    [0,0,0,0,0,0,0],
    [0,1,1,0,0,0,0],
    [0,0,0,1,0,0,0],
    [0,1,1,1,0,0,0],
    [0,0,0,0,0,0,0],
    [0,0,0,0,1,0,0],
    [0,0,0,0,1,0,0],
]

start = (0,0)
goal = (6,6)

pop_size = 100
mut_rate = 0.1
num_g = 1000
elite_rate = 0.1

moves = [(0,1),(1,0),(0,-1),(-1,0)]

def is_vaild_move(grid, position):
    x, y = position
    if 0 <= x < len(grid) and 0 <= y < len(grid[0]) and grid[x][y] == 0:
        return True
    return False

def generate_individual(grid, start, goal, lengh=15):
    position = start
    path=[]
    for _ in range(lengh):
        vaild_moves = [move for move in moves if is_vaild_move(grid, (position[0] + move[0], position[1] + move[1]))]
        if not vaild_moves:
            break
        move = r.choice(vaild_moves)
        path.append(move)
        position = (position[0] + move[0], position[1] + move[1])
    return path

def generate_population(grid, start, goal, size):
    return [generate_individual(grid, start, goal) for _ in range(size)]

def crossover(parent1, parent2):
    crossover_point = len(parent1) // 2
    child1 = parent1[:crossover_point] + parent2[crossover_point:]
    child2 = parent2[:crossover_point] + parent1[crossover_point:]
    return child1, child2

def mutate(individual, rate):
    for i in range(len(individual)):
        if r.random() < rate:
            individual[i] = r.choice(moves)
    return individual

def fitness(grid, start, goal, individual):
    position = start
    for move in individual:
        next_position = (position[0] + move[0], position[1] + move[1])
        if is_vaild_move(grid, next_position):
            position = next_position
    return -abs(position[0] - goal[0]) - abs(position[1]- goal[1])

def select_best_individuals(population, grid, start, goal, elite_rate):
    fitnesses = [(i, fitness(grid, start, goal, individual)) for i, individual in enumerate(population)]
    fitnesses.sort(key=lambda x: x[1], reverse=True)
    return [population[i] for i, _ in fitnesses[:int(elite_rate * len(population))]]

def optimize_path(grid, start, path):
    optimize_path = []
    position = start
    for move in path:
        next_position = (position[0] + move[0], position[1] + move[1])
        if not (0 <= next_position[0] < len(grid) and 0 <= next_position[1] < len(grid[0])) or grid[next_position[0]][next_position[1]] == 1:
            continue
        optimize_path.append(move)
        position = next_position
    return optimize_path

def main(grid, start, goal, pop_size, mut_rate, num_g, elite_rate):
    population = generate_population(grid, start, goal, pop_size)
    best_individual = None
    best_fitness = float("-inf")

    for generation in range(num_g):
        elites = select_best_individuals(population, grid, start, goal, elite_rate)

    new_population = elites.copy()
    while len(new_population) < pop_size:
        parent1 = r.choice(elites)
        parent2 = r.choice(elites)
        child1, child2 = crossover(parent1, parent2)
        new_population.extend([child1, child2])


    for i in range(len(new_population)):
        if i >= len(elites):
            new_population[i] = mutate(new_population[i], mut_rate)

    population = new_population


    current_best_individual = max(population, key=lambda ind: fitness(grid, start, goal, ind))
    current_best_fitness = fitness(grid, start, goal, current_best_individual)
    if current_best_fitness > best_fitness:
        best_fitness = current_best_fitness
        best_individual = current_best_individual

    best_path = optimize_path(grid, start, best_individual)
    return best_path

def display_path(grid, start, goal, path):
    grid_copy = [['x' if cell == 1 else 0 for cell in row] for row in grid]
    position = start
    step = 1
    visited_positions = set()

    for move in path:
        next_position = (position[0] + move[0], position[1] + move[1])
        if is_vaild_move(grid, next_position) and next_position not in visited_positions:
            position = next_position
            visited_positions.add(position)

            if position == goal :
                break

            if grid[position[0]][position[1]] == 0:
                grid_copy[position[0]][position[1]] = step
                step += 1

    grid_copy[start[0]][start[1]] = 'S'
    grid_copy[goal[0]][goal[1]] = 'G'
    for row in grid_copy:
        print(" ".join(str(cell).rjust(2) for cell in row))

if __name__ == "__main__":
    best_path = main(grid_map, start, goal, pop_size, mut_rate, num_g, elite_rate)
    display_path(grid_map, start, goal, best_path)
