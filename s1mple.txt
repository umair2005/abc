# Simple CSP 1
from ortools.sat.python import cp_model

def main():
    model = cp_model.CpModel()

    a = model.new_int_var(0, 3, 'A')
    b = model.new_int_var(0, 3, 'B')
    c = model.new_int_var(0, 3, 'C')
    model.add(a != b)
    model.add(b != c)
    model.add(a + b <= 4)

    solver = cp_model.CpSolver()
    status = solver.Solve(model)

    if status==cp_model.OPTIMAL or status==cp_model.FEASIBLE:
        print(f'A = {solver.value(a)} | B = {solver.value(b)} | C = {solver.value(c)}')
    else:
        print("Solution not found")

main()


# Simple CSP 2
from ortools.sat.python import cp_model

def main():
    model = cp_model.CpModel()

    x = model.new_int_var(0, 20, "x")
    y = model.new_int_var(0, 20, "y")
    z = model.new_int_var(0, 20, "z")

    model.add(x + (2 * y) + z <= 20)
    model.add((3 * x) + y <= 18)
    model.maximize((4 * x) + (2 * y) + z)

    solver = cp_model.CpSolver()
    status = solver.solve(model)

    if status == cp_model.OPTIMAL or status == cp_model.FEASIBLE:
        print(f"Optimal Value = {solver.objective_value}")
        print(f"x = {solver.value(x)} | y = {solver.value(y)} | z = {solver.value(z)}")
    else:
        print("No solution found!")

main()


# CSP Solution Class
from ortools.sat.python import cp_model

class SolutionDisplay(cp_model.CpSolverSolutionCallback):
    def __init__(self, a, b, c):
        cp_model.CpSolverSolutionCallback.__init__(self)
        self.a = a
        self.b = b
        self.c = c
        self.solution_count = 0

    def on_solution_callback(self):
        self.solution_count += 1
        print(f"Solution {self.solution_count}:")
        print(f"A = {self.value(self.a)}, B = {self.value(self.b)}, C = {self.value(self.c)}")


def main():
    model = cp_model.CpModel()
    a = model.new_int_var(0, 3, "A")
    b = model.new_int_var(0, 3, "B")
    c = model.new_int_var(0, 3, "C")

    model.add(a != b)
    model.add(b != c)
    model.add(a + b <= 4)

    solver = cp_model.CpSolver()
    solver.parameters.enumerate_all_solutions = True

    solution_printer = SolutionDisplay(a, b, c)
    status = solver.solve(model, solution_printer)

    print(f"\nTotal Solutions:", solution_printer.solution_count)
    print(f"Status: {solver.status_name(status)}")

main()


# CSP N Queens
import sys
import time
from ortools.sat.python import cp_model

class NQueenSolution(cp_model.CpSolverSolutionCallback):
    def __init__(self, queens: list[cp_model.IntVar]):
        cp_model.CpSolverSolutionCallback.__init__(self)
        self._queens = queens
        self._solution_count = 0
        self._start_time = time.time()

    def solution_count(self):
        return self._solution_count

    def on_solution_callback(self):
        current_time = time.time()
        self._solution_count += 1
        print(f"Solution {self._solution_count}, Time = {current_time - self._start_time} seconds")

        all_queens = range(len(self._queens))
        for i in all_queens:
            for j in all_queens:
                if self.value(self._queens[j]) == i:
                    print("Q", end=" ")
                else:
                    print("_", end=" ")
            print()
        print()

def main():
    model = cp_model.CpModel()
    board_size = 4
    queens = [model.new_int_var(0, board_size - 1, f"x_{i}") for i in range(board_size)]

    model.add_all_different(queens)
    model.add_all_different(queens[i] + i for i in range(board_size))
    model.add_all_different(queens[i] - i for i in range(board_size))

    solver = cp_model.CpSolver()
    solution_printer = NQueenSolution(queens)
    solver.parameters.enumerate_all_solutions = True
    solver.solve(model, solution_printer)
    print(f"Solutions found: {solution_printer.solution_count()}")

main()


# CSP Additional Constraints
from ortools.sat.python import cp_model

model = cp_model.CpModel()
x = model.NewIntVar(0, 10, "x")
y = model.NewIntVar(0, 10, "y")
z = model.NewIntVar(0, 10, "z")

# Applying constraints
model.Add(x + y <= 10)
model.AddAllDifferent([x, y, z])
model.AddAllowedAssignments([x, y], [(1, 2), (3, 4)])
model.AddForbiddenAssignments([x, y], [(1, 1), (2, 2)])

# Additional constraints
target = model.NewIntVar(0, 10, "target")
model.AddMinEquality(target, [x, y, z])
model.AddMaxEquality(target, [x, y, z])

num = model.NewIntVar(1, 10, "num")
denom = model.NewIntVar(1, 5, "denom")
div_target = model.NewIntVar(0, 10, "div_target")
model.AddDivisionEquality(div_target, num, denom)

factors = [model.NewIntVar(1, 5, f"f{i}") for i in range(2)]
prod_target = model.NewIntVar(1, 25, "prod_target")
model.AddMultiplicationEquality(prod_target, factors)

solver = cp_model.CpSolver()
status = solver.Solve(model)
if status == cp_model.OPTIMAL or status == cp_model.FEASIBLE:
    print(f"x = {solver.Value(x)}, y = {solver.Value(y)}, z = {solver.Value(z)}")
    print(f"Min target = {solver.Value(target)}, Max target = {solver.Value(target)}")
    print(f"Division result = {solver.Value(div_target)}, Product result = {solver.Value(prod_target)}")
else:
    print("No feasible solution found.")


# Simple Minimax
import math

class Node:
    def __init__(self, value = None):
        self.value = value
        self.children = []
        self.minmax_value = None
        self.best_child = None


def compute_minimax(node, depth, maximizing_player, order_list):
    order_list.append(node.value)
    
    if depth == 0 or not node.children:
        return node.value

    if maximizing_player:
        value = -math.inf
        for child in node.children:
            child_value = compute_minimax(child, depth - 1, False, order_list)
            
            if child_value > value:
                value = child_value
                node.best_child = child
                
    else:
        value = math.inf
        for child in node.children:
            child_value = compute_minimax(child, depth - 1, True, order_list)
            
            if child_value < value:
                value = child_value
                node.best_child = child

    node.minmax_value = value
    return value


def print_optimal_path(root):
    path = []
    current = root

    while current:
        path.append(current.value)
        current = current.best_child

    print("Optimal Path:", " -> ".join(map(str, path)))


def main():
    leaf4 = Node(4)
    leaf7 = Node(7) 
    leaf2 = Node(2) 
    leaf5 = Node(5)
    leaf1 = Node(1) 
    leaf8 = Node(8) 
    leaf3 = Node(3) 
    leaf6 = Node(6)

    n3 = Node("N3") 
    n3.children = [leaf4, leaf7]
    n4 = Node("N4")
    n4.children = [leaf2, leaf5]
    n5 = Node("N5")
    n5.children = [leaf1, leaf8]
    n6 = Node("N6") 
    n6.children = [leaf3, leaf6]

    n1 = Node("N1")
    n1.children = [n3, n4]
    n2 = Node("N2")
    n2.children = [n5, n6]

    root = Node("Root")
    root.children = [n1, n2]

    order_full = []
    root_minimax = compute_minimax(root, 3, True, order_full)

    print("-------------- Full Minimax (depth = 3) --------------")
    print("Minimax values:")
    print(f"Root: {root.minmax_value}")
    print(f"N1: {n1.minmax_value}")
    print(f"N2: {n2.minmax_value}")
    print(f"N3: {n3.minmax_value}")
    print(f"N4: {n4.minmax_value}")
    print(f"N5: {n5.minmax_value}")
    print(f"N6: {n6.minmax_value}")
    print(f"Order of visited nodes: {order_full}")
    
    print_optimal_path(root)

    n3_dl = Node(5.5)
    n4_dl = Node(3.5)
    n5_dl = Node(4.5)
    n6_dl = Node(4.5)

    n1_dl = Node("N1") 
    n1_dl.children = [n3_dl, n4_dl]
    n2_dl = Node("N2")
    n2_dl.children = [n5_dl, n6_dl]

    root_dl = Node("Root")
    root_dl.children = [n1_dl, n2_dl]

    order_limit = []
    compute_minimax(root_dl, 2, True, order_limit)

    print("\n-------------- Depth-Limited Minimax (depth = 2) --------------")
    print("Minimax values:")
    print(f"Root: {root_dl.minmax_value}")
    print(f"N1: {n1_dl.minmax_value}")
    print(f"N2: {n2_dl.minmax_value}")
    print(f"N3: {n3_dl.value}")
    print(f"N4: {n4_dl.value}")
    print(f"N5: {n5_dl.value}")
    print(f"N6: {n6_dl.value}")
    print(f"Order of visited nodes: {order_limit}")
    
    print_optimal_path(root_dl)

main()


# Agent Class Minimax
import math

class Node:
    def __init__(self, value=None):
        self.value = value
        self.children = []
        self.minmax_value = None

class MinimaxAgent:
    def __init__(self, depth):
        self.depth = depth

    def formulate_goal(self, node):
        return "Goal reached" if node.minmax_value is not None else "Searching"

    def act(self, node, environment):
        goal_status = self.formulate_goal(node)
        if goal_status == "Goal reached":
            return f"Minimax value for root node: {node.minmax_value}"
        else:
            return environment.compute_minimax(node, self.depth)

class Environment:
    def __init__(self, tree):
        self.tree = tree
        self.computed_nodes = []

    def get_percept(self, node):
        return node

    def compute_minimax(self, node, depth, maximizing_player=True):
        if depth == 0 or not node.children:
            self.computed_nodes.append(node.value)
            return node.value

        if maximizing_player:
            value = -math.inf
            for child in node.children:
                child_value = self.compute_minimax(child, depth - 1, False)
                value = max(value, child_value)
            node.minmax_value = value
            self.computed_nodes.append(node.value)
            return value
        else:
            value = math.inf
            for child in node.children:
                child_value = self.compute_minimax(child, depth - 1, True)
                value = min(value, child_value)
            node.minmax_value = value
            self.computed_nodes.append(node.value)
            return value
        
def run_agent(agent, environment, start_node):
    percept = environment.get_percept(start_node)
    agent.act(percept, environment)

root = Node('A')
n1 = Node('B')
n2 = Node('C')
root.children = [n1, n2]

n3 = Node('D')
n4 = Node('E')
n5 = Node('F')
n6 = Node('G')
n1.children = [n3, n4]
n2.children = [n5, n6]

n7 = Node(2)
n8 = Node(3)
n9 = Node(5)
n10 = Node(9)
n3.children = [n7, n8]
n4.children = [n9, n10]

n11 = Node(0)
n12 = Node(1)
n13 = Node(7)
n14 = Node(5)
n5.children = [n11, n12]
n6.children = [n13, n14]

depth = 3
agent = MinimaxAgent(depth)
environment = Environment(root)
run_agent(agent, environment, root)
print("Computed Nodes:", environment.computed_nodes)
print("Minimax values:")
print("A:", root.minmax_value)
print("B:", n1.minmax_value)
print("C:", n2.minmax_value)
print("D:", n3.minmax_value)
print("E:", n4.minmax_value)
print("F:", n5.minmax_value)
print("G:", n6.minmax_value)


# Simple Alpha Beta Pruning
import math

class Node:
    def __init__(self, value=None):
        self.value = value
        self.children = []
        self.minmax_value = None

class Environment:
    def __init__(self):
        self.computed_nodes = []
        self.pruned_nodes = []

    def alpha_beta(self, node, depth, alpha, beta, maximizing_player = True):
        self.computed_nodes.append(node.value)
        if depth == 0 or not node.children:
            return node.value

        if maximizing_player:
            value = -math.inf
            for i, child in enumerate(node.children):
                value = max(value, self.alpha_beta(child, depth - 1, alpha, beta, False))
                alpha = max(alpha, value)
                if beta <= alpha:
                    for skipped in node.children[i + 1:]:
                        self.pruned_nodes.append(skipped.value)
                    break
        else:
            value = math.inf
            for i, child in enumerate(node.children):
                value = min(value, self.alpha_beta(child, depth - 1, alpha, beta, True))
                beta = min(beta, value)
                if beta <= alpha:
                    for skipped in node.children[i + 1:]:
                        self.pruned_nodes.append(skipped.value)
                    break

        node.minmax_value = value
        return value


def main():
    root = Node('Root')
    n1 = Node('N1')
    n2 = Node('N2')
    n3 = Node('N3')
    n4 = Node('N4')
    n5 = Node('N5')
    n6 = Node('N6')

    root.children = [n1, n2]
    n1.children = [n3, n4]
    n2.children = [n5, n6]
    n3.children = [Node(4), Node(7)]
    n4.children = [Node(2), Node(5)]
    n5.children = [Node(1), Node(8)]
    n6.children = [Node(3), Node(6)]

    env = Environment()
    env.alpha_beta(root, 3, -math.inf, math.inf, True)

    print("-------------- Minimax Values --------------")
    print(f"Root: {root.minmax_value}")
    print(f"N1: {n1.minmax_value}")
    print(f"N2: {n2.minmax_value}")
    print(f"N3: {n3.minmax_value}")
    print(f"N4: {n4.minmax_value}")
    print(f"N5: {n5.minmax_value}")
    print(f"N6: {n6.minmax_value}")
    print(f"\nVisited Nodes: {env.computed_nodes}")
    print(f"Pruned Nodes: {env.pruned_nodes}")

main()


# Agent Class Alpha Beta Pruning
import math

class Node:
    def __init__(self, value=None):
        self.value = value
        self.children = []
        self.minmax_value = None

class MinimaxAgent:
    def __init__(self, depth):
        self.depth = depth

    def formulate_goal(self, node):
        return "Goal reached" if node.minmax_value is not None else "Searching"

    def act(self, node, environment):
        goal_status = self.formulate_goal(node)
        if goal_status == "Goal reached":
            return f"Minimax value for root node: {node.minmax_value}"
        else:
            return environment.alpha_beta_search(node, self.depth, -math.inf, math.inf, True)
        
class Environment:
    def __init__(self, tree):
        self.tree = tree
        self.computed_nodes = []

    def get_percept(self, node):
        return node

    def alpha_beta_search(self, node, depth, alpha, beta, maximizing_player=True):
        self.computed_nodes.append(node.value)
        if depth == 0 or not node.children:
            return node.value

        if maximizing_player:
            value = -math.inf
            for child in node.children:
                value = max(value, self.alpha_beta_search(child, depth - 1, alpha, beta, False))
                alpha = max(alpha, value)
                if beta <= alpha:
                    print("Pruned node:", child.value)
                    break
            node.minmax_value = value
            return value
        else:
            value = math.inf
            for child in node.children:
                value = min(value, self.alpha_beta_search(child, depth - 1, alpha, beta, True))
                beta = min(beta, value)
                if beta <= alpha:
                    print("Pruned node:", child.value)
                    break
            node.minmax_value = value
            return value
        
def run_agent(agent, environment, start_node):
    percept = environment.get_percept(start_node)
    agent.act(percept, environment)

root = Node('A')
n1 = Node('B')
n2 = Node('C')
root.children = [n1, n2]

n3 = Node('D')
n4 = Node('E')
n5 = Node('F')
n6 = Node('G')
n1.children = [n3, n4]
n2.children = [n5, n6]

n7 = Node(2)
n8 = Node(3)
n9 = Node(5)
n10 = Node(9)
n3.children = [n7, n8]
n4.children = [n9, n10]

n11 = Node(0)
n12 = Node(1)
n13 = Node(7)
n14 = Node(5)
n5.children = [n11, n12]
n6.children = [n13, n14]

depth = 3
agent = MinimaxAgent(depth)
environment = Environment(root)
run_agent(agent, environment, root)
print("Computed Nodes:", environment.computed_nodes)
print("Minimax values:")
print(f"A: {root.minmax_value}")
print(f"B: {n1.minmax_value}")
print(f"C: {n2.minmax_value}")
print(f"D: {n3.minmax_value}")
print(f"E: {n4.minmax_value}")
print(f"F: {n5.minmax_value}")
print(f"G: {n6.minmax_value}")


# Simple Probability
import random

suits = ['Hearts', 'Diamonds', 'Clubs', 'Spades']
ranks = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'Jack', 'Queen', 'King', 'Ace']
deck = [(rank, suit) for suit in suits for rank in ranks]
print(deck)
face_cards = [card for card in deck if card[0] in ['Jack', 'Queen', 'King']]

trials = 100000
king_given_face = 0
for _ in range(trials):
    card = random.choice(face_cards)
    if card[0] == 'King':
        king_given_face += 1

simulated_probability = king_given_face / trials

theoretical_probability = 4 / 12

print(f"Theoretical P(King | Face card): {theoretical_probability:.3f}")
print(f"Simulated P(King | Face card):   {simulated_probability:.3f}")


# Bayesian Networks 1
from pgmpy.models import DiscreteBayesianNetwork
from pgmpy.factors.discrete import TabularCPD
from pgmpy.inference import VariableElimination

model = DiscreteBayesianNetwork([
    ('Burglary', 'Alarm'),
    ('Earthquake', 'Alarm'),
    ('Alarm', 'David_calls'),
    ('Alarm', 'Sophia_calls')
])

cpd_burglary = TabularCPD(
    variable='Burglary',
    variable_card=2,
    values=[[0.999], [0.001]],  # [False, True]
    state_names={'Burglary': ['False', 'True']}
)

cpd_earthquake = TabularCPD(
    variable='Earthquake',
    variable_card=2,
    values=[[0.998], [0.002]],  # [False, True]
    state_names={'Earthquake': ['False', 'True']}
)

cpd_alarm = TabularCPD(
    variable='Alarm',
    variable_card=2,
    values=[
        [0.999,   0.71,   0.06,   0.05],  # Alarm=False
        [0.001,   0.29,   0.94,   0.95]   # Alarm=True
    ],
    evidence=['Burglary', 'Earthquake'],
    evidence_card=[2, 2],
    state_names={
        'Alarm': ['False', 'True'],
        'Burglary': ['False', 'True'],
        'Earthquake': ['False', 'True']
    }
)

cpd_david = TabularCPD(
    variable='David_calls',
    variable_card=2,
    values=[
        # A=F    A=T
        [0.95,  0.1],  # David_calls=False
        [0.05,  0.9]   # David_calls=True
    ],
    evidence=['Alarm'],
    evidence_card=[2],
    state_names={
        'David_calls': ['False', 'True'],
        'Alarm': ['False', 'True']
    }
)

cpd_sophia = TabularCPD(
    variable='Sophia_calls',
    variable_card=2,
    values=[
        [0.99,  0.3],  # Sophia_calls=False
        [0.01,  0.7]   # Sophia_calls=True
    ],
    evidence=['Alarm'],
    evidence_card=[2],
    state_names={
        'Sophia_calls': ['False', 'True'],
        'Alarm': ['False', 'True']
    }
)

model.add_cpds(cpd_burglary, cpd_earthquake, cpd_alarm, cpd_david, cpd_sophia)
assert model.check_model()
infer = VariableElimination(model)

# Calculate P(Alarm=True | Burglary=True)
prob_alarm_given_b_true = infer.query(
    variables=['Alarm'],
    evidence={'Burglary': 'True'}
)
print("P(Alarm=True | Burglary=True):", prob_alarm_given_b_true.values[1])  # Index 1 is 'True'

# Calculate P(Alarm=True | Burglary=False)
prob_alarm_given_b_false = infer.query(
    variables=['Alarm'],
    evidence={'Burglary': 'False'}
)
print("P(Alarm=True | Burglary=False):", prob_alarm_given_b_false.values[1])  # Index 1 is 'True'

# Calculate P(Burglary=True | Alarm=True)
prob_burglary_given_a_true = infer.query(
    variables=['Burglary'],
    evidence={'Alarm': 'True'}
)
print("P(Burglary=True | Alarm=True):", prob_burglary_given_a_true.values[1])  # Index 1 is 'True'


# Bayesian Networks 2
from pgmpy.models import BayesianModel
from pgmpy.factors.discrete import TabularCPD
from pgmpy.inference import VariableElimination

model = BayesianModel([
    ('Intelligence', 'Grade'),
    ('StudyHours', 'Grade'),
    ('Difficulty', 'Grade'),
    ('Grade', 'Pass')
])

cpd_I = TabularCPD(variable = 'Intelligence', variable_card = 2, values = [[0.7], [0.3]], state_names = {'Intelligence': ['High', 'Low']})
cpd_S = TabularCPD(variable = 'StudyHours', variable_card = 2, values = [[0.6], [0.4]], state_names = {'StudyHours': ['Sufficient', 'Insufficient']})
cpd_D = TabularCPD(variable = 'Difficulty', variable_card = 2, values = [[0.4], [0.6]], state_names = {'Difficulty': ['Hard', 'Easy']})

cpd_G = TabularCPD(
    variable = 'Grade', 
    variable_card = 3,
    values = [
        [0.7, 0.5, 0.4, 0.2, 0.5, 0.3, 0.2, 0.1], # A
        [0.2, 0.3, 0.4, 0.5, 0.3, 0.4, 0.4, 0.3], # B
        [0.1, 0.2, 0.2, 0.3, 0.2, 0.3, 0.4, 0.6]  # C
    ],
    evidence = ['Intelligence', 'StudyHours', 'Difficulty'],
    evidence_card = [2, 2, 2],
    state_names = {
        'Grade': ['A', 'B', 'C'],
        'Intelligence': ['High', 'Low'],
        'StudyHours': ['Sufficient', 'Insufficient'],
        'Difficulty': ['Hard', 'Easy']
    }
)

cpd_P = TabularCPD(
    variable = 'Pass',
    variable_card = 2,
    values = [
        [0.95, 0.80, 0.50], # Yes
        [0.05, 0.20, 0.50]  # No
    ],
    evidence = ['Grade'],
    evidence_card = [3],
    state_names = {
        'Pass': ['Yes', 'No'],
        'Grade': ['A', 'B', 'C']
    }
)

model.add_cpds(cpd_I, cpd_S, cpd_D, cpd_G, cpd_P)
print("Model valid:", model.check_model())
inference = VariableElimination(model)

result1 = inference.query(
    variables = ['Pass'],
    evidence = {'StudyHours': 'Sufficient', 'Difficulty': 'Hard'}
)

print("\nQuery 1: P(Pass | StudyHours = Sufficient, Difficulty = Hard):")
print(result1)

result2 = inference.query(
    variables = ['Intelligence'],
    evidence = {'Pass': 'Yes'}
)

print("\nQuery 2: P(Intelligence | Pass = Yes):")
print(result2)


# Hidden Markov Problem 1
import numpy as np

states = ["Red", "Blue"]
transition_matrix = np.array([[0.5, 0.5],  # From Red -> Red or Blue
                              [0.5, 0.5]]) # From Blue -> Red or Blue

def simulate_markov_process(initial_state, num_steps):
    current_state = initial_state
    state_sequence = [current_state]

    for _ in range(num_steps):
        if current_state == "Red":
            next_state = np.random.choice(states, p=transition_matrix[0])
        else:
            next_state = np.random.choice(states, p=transition_matrix[1])

        state_sequence.append(next_state)
        current_state = next_state

    return state_sequence

initial_state = "Red"
num_steps = 10
state_sequence = simulate_markov_process(initial_state, num_steps)

print(f"State sequence for {num_steps} steps starting from {initial_state}:")
print(" -> ".join(state_sequence))


# Hidden Markov Problem 2
import numpy as np

states = ["Sunny", "Cloudy", "Rainy"]
transition_matrix = np.array([
    [0.6, 0.3, 0.1],
    [0.3, 0.4, 0.3],
    [0.2, 0.3, 0.5]
])

def simulate_weather(days):
    current_state = 0
    sequence = [states[current_state]]

    for _ in range(days - 1):
        next_state = np.random.choice([0, 1, 2], p = transition_matrix[current_state])
        sequence.append(states[next_state])
        current_state = next_state

    return sequence

def count_rainy_days(sequence):
    return sequence.count("Rainy")

weather_sequence = simulate_weather(10)
print("Weather for 10 days:")
print(weather_sequence)

rainy_days = count_rainy_days(weather_sequence)
print("\nNumber of Rainy Days:", rainy_days)

simulations = 10000
count_atleast_3_rainy = 0

for _ in range(simulations):
    seq = simulate_weather(10)
    if count_rainy_days(seq) >= 3:
        count_atleast_3_rainy += 1

probability = count_atleast_3_rainy / simulations
print("\nProbability of at least 3 rainy days in 10 days:")
print(probability)


# Logistic Regression 
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

data = load_iris()
X = data.data
y = data.target

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

model = LogisticRegression(max_iter=200)

model.fit(X_train, y_train)

y_pred = model.predict(X_test)

accuracy = accuracy_score(y_test, y_pred)
print(f'Accuracy: {accuracy}')


# SVM
from sklearn import datasets
from sklearn.svm import SVC
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

iris = datasets.load_iris()
X = iris.data
y = iris.target
y = (y == 0).astype(int)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)
svm = SVC(kernel='rbf', C=1, gamma='scale')
svm.fit(X_train, y_train)
y_pred = svm.predict(X_test)
print("SVM Accuracy:", accuracy_score(y_test, y_pred))


# Linear Regression
import numpy as np
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error

x_train, x_test, y_train, y_test = train_test_split(x, y, test_size=0.2, random_state=42)

LR = LinearRegression()
ModelLR = LR.fit(x_train, y_train)

PredictionLR = ModelLR.predict(x_test)
print("Predictions:", PredictionLR)

from sklearn.metrics import r2_score
print("===================LR Testing Accuracy================")
teachLR = r2_score(y_test, PredictionLR)
testingAccLR = teachLR * 100
print(testingAccLR)


# Decision Tree
from sklearn.tree import DecisionTreeClassifier

DT = DecisionTreeClassifier()
ModelDT = DT.fit(x_train, y_train)

PredictionDT = DT.predict(x_test)
print("Predictions:", PredictionDT)

print("====================DT Training Accuracy===============")
tracDT = DT.score(x_train, y_train) # The score method gives accuracy directly
TrainingAccDT = tracDT * 100
print(f"Training Accuracy: {TrainingAccDT:.2f}%")

# Model Testing Accuracy
print("=====================DT Testing Accuracy=================")
teacDT = accuracy_score(y_test, PredictionDT)
testingAccDT = teacDT * 100
print(f"Testing Accuracy: {testingAccDT:.2f}%")


# Naive Bayes
from sklearn.model_selection import train_test_split
from sklearn.naive_bayes import GaussianNB
from sklearn.metrics import accuracy_score
import numpy as np

X = np.random.rand(100, 5)
y = np.random.randint(0, 2, 100)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
model = GaussianNB()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
print(f"Model Accuracy: {accuracy:.2f}")


# LOOCV
from sklearn.model_selection import LeaveOneOut

loo = LeaveOneOut()
loo_scores = []

for train_index, test_index in loo.split(X):
    X_train, X_test = X.iloc[train_index], X.iloc[test_index]
    y_train, y_test = y.iloc[train_index], y.iloc[test_index]

    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    loo_scores.append(accuracy_score(y_test, y_pred))

print("LOOCV Average Accuracy:", np.mean(loo_scores))


# ROC Curve
from sklearn.metrics import roc_curve, roc_auc_score
import matplotlib.pyplot as plt
probabilities = DT.predict_proba(x_test)[:, 1]

fpr, tpr, thresholds = roc_curve(y_test, probabilities)
roc_auc = roc_auc_score(y_test, probabilities)

plt.figure(figsize=(8, 6))
plt.plot(fpr, tpr, color='blue', lw=2, label=f'ROC Curve (AUC = {roc_auc:.2f})')
plt.fill_between(fpr, tpr, color='skyblue', alpha=0.4)
plt.plot([0, 1], [0, 1], color='gray', linestyle='--')
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve with AUC Area')
plt.legend(loc='lower right')
plt.show()


# K-Fold
from sklearn.model_selection import KFold
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score
import numpy as np
import pandas as pd
import seaborn as sns

df = sns.load_dataset("titanic")

X = df[['age', 'fare']].fillna(df[['age', 'fare']].mean())
y = df['survived']

X = pd.DataFrame(X)
y = pd.Series(y)

kf = KFold(n_splits=5, shuffle=True, random_state=42)

model = LogisticRegression()
accuracy_scores = []

for train_index, test_index in kf.split(X):
    X_train, X_test = X.iloc[train_index], X.iloc[test_index]
    y_train, y_test = y.iloc[train_index], y.iloc[test_index]

    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    acc = accuracy_score(y_test, y_pred)
    accuracy_scores.append(acc)

print("K-Fold CV Average Accuracy:", np.mean(accuracy_scores))


# Lab 10 LT1
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import r2_score, mean_squared_error

housing_data = pd.read_csv("/content/sample_data/california_housing_test.csv")
housing_data = housing_data.dropna()

features = housing_data.drop("median_house_value", axis=1)
target = housing_data["median_house_value"]

X_train, X_test, y_train, y_test = train_test_split(features, target, test_size=0.2, random_state=42)
reg_model = LinearRegression()
reg_model.fit(X_train, y_train)

predictions = reg_model.predict(X_test)
print(f"R2 Score: {r2_score(y_test, predictions)}")

rmse_value = np.sqrt(mean_squared_error(y_test, predictions))
print(f"RMSE: {rmse_value}")

sample_input = pd.DataFrame([features.iloc[0]], columns=features.columns)
print(f"Predicted Price: {reg_model.predict(sample_input)}")


# Lab 10 LT2
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, classification_report

spam_data = pd.read_csv("/content/spam.csv", encoding='latin1')
spam_data = spam_data[["v1", "v2"]]
spam_data.columns = ["label", "message"]

spam_data["label"] = spam_data["label"].map({"ham": 0, "spam": 1})
texts = spam_data["message"]
labels = spam_data["label"]

vectorizer = CountVectorizer(stop_words="english")
X_features = vectorizer.fit_transform(texts)
X_train, X_test, y_train, y_test = train_test_split(X_features, labels, test_size=0.2, random_state=42)

log_model = LogisticRegression(max_iter=1000)
log_model.fit(X_train, y_train)

predictions = log_model.predict(X_test)

print("Accuracy:", accuracy_score(y_test, predictions))
print("\nClassification Report:\n", classification_report(y_test, predictions))

sample_message = ["Congratulations! You have won a free iPhone. Claim now!!!"]
sample_vector = vectorizer.transform(sample_message)

result = log_model.predict(sample_vector)
print("\nPrediction:", result[0])
print("0 = Not Spam, 1 = Spam")


# Lab 10 LT3
import pandas as pd
import numpy as np

from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report

df = pd.read_csv("/content/customer_data.csv")
df = df.dropna()

if "name" in df.columns:
    df = df.drop("name", axis=1)

numeric_cols = df.select_dtypes(include=np.number).columns

for col in numeric_cols:
    lower = df[col].quantile(0.01)
    upper = df[col].quantile(0.99)
    df[col] = np.clip(df[col], lower, upper)

cat_cols = [c for c in ["gender", "education", "country"] if c in df.columns]
df = pd.get_dummies(df, columns=cat_cols, drop_first=True)

df["Customer_Type"] = (df["spending"] > df["spending"].median()).astype(int)

X = df.drop("Customer_Type", axis=1)
y = df["Customer_Type"]

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y, test_size=0.2, random_state=42)

model = SVC(kernel="linear")
model.fit(X_train, y_train)

y_pred = model.predict(X_test)

print("Accuracy:", accuracy_score(y_test, y_pred))
print("\nConfusion Matrix:\n", confusion_matrix(y_test, y_pred))
print("\nClassification Report:\n", classification_report(y_test, y_pred))

print("\nHyperplane Weights (feature importance direction):")
print(model.coef_)

print("\nIntercept:")
print(model.intercept_)

print("\nRule Insight:")
print("If weighted sum of features > threshold → High Value Customer (1)")
print("Else → Low Value Customer (0)")


# K means Clustering
import numpy as nm
import matplotlib.pyplot as mtp
import pandas as pd
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

df = pd.read_csv('Mall_Customers.csv')
df.head()

x = df.iloc[:, [3, 4]].values

from sklearn.cluster import KMeans
wcss_list= []

for i in range(1, 11):
    kmeans = KMeans(n_clusters=i, init='k-means++', random_state= 42)
    kmeans.fit(x)
    wcss_list.append(kmeans.inertia_)

mtp.plot(range(1, 11), wcss_list)
mtp.title('The Elobw Method Graph')
mtp.xlabel('Number of clusters(k)')
mtp.ylabel('wcss_list')
mtp.show()

scaler = StandardScaler()
X_scaled = scaler.fit_transform(x)

kmeans = KMeans(n_clusters=5, init='k-means++', random_state= 42)
y_predict= kmeans.fit_predict(X_scaled)

#visulaizing the clusters
mtp.scatter(x[y_predict == 0, 0], x[y_predict == 0, 1], s = 100, c = 'blue', label = 'Cluster 1') #for first cluster
mtp.scatter(x[y_predict == 1, 0], x[y_predict == 1, 1], s = 100, c = 'green', label = 'Cluster 2') #for second cluster
mtp.scatter(x[y_predict== 2, 0], x[y_predict == 2, 1], s = 100, c = 'red', label = 'Cluster 3') #for third cluster
mtp.scatter(x[y_predict == 3, 0], x[y_predict == 3, 1], s = 100, c = 'black', label = 'Cluster 4') #for fourth cluster
mtp.scatter(x[y_predict == 4, 0], x[y_predict == 4, 1], s = 100, c = 'purple', label = 'Cluster 5') #for fifth cluster
mtp.scatter(kmeans.cluster_centers_[:, 0], kmeans.cluster_centers_[:,1], s = 300, c = 'yellow', label = 'Centroid')
mtp.title('Clusters of customers')
mtp.xlabel('Annual Income (k$)')
mtp.ylabel('Spending Score (1-100)')
mtp.legend()
mtp.show()


# K Means LT1
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

mall_data = pd.read_csv("Mall_Customers.csv")
customer_features = mall_data.drop("CustomerID", axis=1)
customer_features = pd.get_dummies(customer_features, drop_first=True)

kmeans_raw = KMeans(n_clusters=5, random_state=42)
labels_raw = kmeans_raw.fit_predict(customer_features)

plt.figure(figsize=(6,4))
plt.scatter(customer_features.iloc[:, 0], customer_features.iloc[:, 1], c=labels_raw)
plt.title("Clustering Without Scaling")
plt.show()

scaled_features = customer_features.copy()
scaler = StandardScaler()
scale_cols = scaled_features.columns.drop("Age")

scaled_features[scale_cols] = scaler.fit_transform(scaled_features[scale_cols])
kmeans_scaled = KMeans(n_clusters=5, random_state=42)
labels_scaled = kmeans_scaled.fit_predict(scaled_features)

plt.figure(figsize=(6,4))
plt.scatter(scaled_features.iloc[:, 0], scaled_features.iloc[:, 1], c=labels_scaled)
plt.title("Clustering With Scaling")
plt.show()


# K Means LT2
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

student_data = {
    'id': [1,2,3,4,5,6,7,8,9,10],
    'gpa': [3.5,2.8,3.9,1.8,2.5,3.2,3.7,2.0,2.9,3.8],
    'hours_studied': [20,10,25,5,12,18,22,7,15,24],
    'attendance': [90,70,95,60,75,85,92,65,80,96]
}

students_df = pd.DataFrame(student_data)
features = students_df[['gpa', 'hours_studied', 'attendance']]
scaler = StandardScaler()
scaled_features = scaler.fit_transform(features)

inertia_values = []
k_range = range(2, 7)

for k in k_range:
    model = KMeans(n_clusters=k, random_state=42)
    model.fit(scaled_features)
    inertia_values.append(model.inertia_)

plt.plot(k_range, inertia_values)
plt.title("Elbow Method")
plt.xlabel("Number of Clusters (K)")
plt.ylabel("WCSS")
plt.show()

final_model = KMeans(n_clusters=3, random_state=42)
cluster_labels = final_model.fit_predict(scaled_features)
students_df['Cluster'] = cluster_labels

plt.scatter(students_df['hours_studied'], students_df['gpa'], c=cluster_labels)
plt.xlabel("Hours Studied")
plt.ylabel("GPA")
plt.title("Student Clusters")

plt.tight_layout()
plt.show()
print(students_df.to_string(index=False))


# EDA
# 1. Load data
df = pd.read_csv("data.csv")

# 2. Structure
df.info()
df.describe()

# 3. Missing values
print(df.isnull().sum())

# 4. Handle missing
df.fillna(method='ffill', inplace=True)

# 5. Duplicates
print(df.duplicated().sum())
df.drop_duplicates(inplace=True)

df.head()
df.shape

df.info()
df.describe()          # numeric columns
df.describe(include='all')   # includes categorical

df['column_name'].unique()
df['column_name'].value_counts()

df.isnull().sum()
df.isnull().sum().sort_values(ascending=False)

(df.isnull().sum() / len(df)) * 100

df.dropna()              # drop rows with missing values
df.dropna(axis=1)        # drop columns
df.fillna(0)             # fill with 0
df.fillna(df.mean(numeric_only=True))   # fill numeric with mean
df['col'].fillna(df['col'].mode()[0], inplace=True)  # categorical

df.duplicated().sum()
df[df.duplicated()]
df.drop_duplicates(inplace=True)

(df == 0).sum()

df.dtypes

df['col'] = df['col'].astype(int)
df['date'] = pd.to_datetime(df['date'])