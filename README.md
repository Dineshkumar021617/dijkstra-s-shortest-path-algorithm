# Dijkstra's Shortest Path Algorithm
## AIM

To develop a code to find the shortest route from the source to the destination point using Dijkstra's shortest path algorithm.

## THEORY:
Best-first search algorithm always selects the path which appears best at that moment. It is the combination of depth-first search and breadth-first search algorithms. Best-first search allows us to take the advantages of both algorithms. With the help of best-first search, at each step, we can choose the most promising node. In the best first search algorithm, we expand the node which is closest to the goal node. The best first search uses the concept of a priority queue. It is a search algorithm that works on a specific rule. The aim is to reach the goal from the initial state via the shortest path. Best First Search is an algorithm for finding the shortest path from a given starting node to a goal node in a graph. The algorithm works by expanding the nodes of the graph in order of increasing the distance from the starting node until the goal node is reached.

## DESIGN STEPS

### STEP 1:
Identify a location in the google map:

### STEP 2:
Select a specific number of nodes with distance

### STEP 3:
Create a dictionary with all the node pairs (keys) and their respective distances as the values.

### Step 4:
Implement the search algorithm by passing any node and f(node) to find the Best route.

### Step 5:
Display the route sequence.


## ROUTE MAP
#### Include your own map
![Map](https://user-images.githubusercontent.com/75234807/167794231-accea352-a7a6-4e58-a490-bd511616a193.png)


## PROGRAM:
```python
%matplotlib inline
import matplotlib.pyplot as plt
import random
import math
import sys
from collections import defaultdict, deque, Counter
from itertools import combinations
import heapq
class Problem(object):
    """The abstract class for a formal problem. A new domain subclasses this,
    overriding `actions` and `results`, and perhaps other methods.
    The default heuristic is 0 and the default action cost is 1 for all states.
    When yiou create an instance of a subclass, specify `initial`, and `goal` states 
    (or give an `is_goal` method) and perhaps other keyword args for the subclass."""

    def __init__(self, initial=None, goal=None, **kwds): 
        self.__dict__.update(initial=initial, goal=goal, **kwds) 
        
    def actions(self, state):        
        raise NotImplementedError
    def result(self, state, action): 
        raise NotImplementedError
    def is_goal(self, state):        
        return state == self.goal
    def action_cost(self, s, a, s1): 
        return 1
    
    def __str__(self):
        return '{0}({1}, {2})'.format(
            type(self).__name__, self.initial, self.goal)
class Node:
    "A Node in a search tree."
    def __init__(self, state, parent=None, action=None, path_cost=0):
        self.__dict__.update(state=state, parent=parent, action=action, path_cost=path_cost)

    def __str__(self): 
        return '<{0}>'.format(self.state)
    def __len__(self): 
        return 0 if self.parent is None else (1 + len(self.parent))
    def __lt__(self, other): 
        return self.path_cost < other.path_cost
failure = Node('failure', path_cost=math.inf) # Indicates an algorithm couldn't find a solution.
cutoff  = Node('cutoff',  path_cost=math.inf) # Indicates iterative deepening search was cut off.
def expand(problem, node):
    "Expand a node, generating the children nodes."
    s = node.state
    for action in problem.actions(s):
        s1 = problem.result(s, action)
        cost = node.path_cost + problem.action_cost(s, action, s1)
        yield Node(s1, node, action, cost)
        

def path_actions(node):
    "The sequence of actions to get to this node."
    if node.parent is None:
        return []  
    return path_actions(node.parent) + [node.action]


def path_states(node):
    "The sequence of states to get to this node."
    if node in (cutoff, failure, None): 
        return []
    return path_states(node.parent) + [node.state]
class PriorityQueue:
    """A queue in which the item with minimum f(item) is always popped first."""

    def __init__(self, items=(), key=lambda x: x): 
        self.key = key
        self.items = [] # a heap of (score, item) pairs
        for item in items:
            self.add(item)
         
    def add(self, item):
        """Add item to the queuez."""
        pair = (self.key(item), item)
        heapq.heappush(self.items, pair)

    def pop(self):
        """Pop and return the item with min f(item) value."""
        return heapq.heappop(self.items)[1]
    
    def top(self): return self.items[0][1]

    def __len__(self): return len(self.items)

def best_first_search(problem, f):
    "Search nodes with minimum f(node) value first."
    node = Node(problem.initial)
    frontier = PriorityQueue([node], key=f)
    reached = {problem.initial: node}
    while frontier:
        node = frontier.pop()
        if problem.is_goal(node.state):
            return node
        for child in expand(problem, node):
            s = child.state
            if s not in reached or child.path_cost < reached[s].path_cost:
                reached[s] = child
                frontier.add(child)
    return failure

def g(n): 
    return n.path_cost
class RouteProblem(Problem):
    """A problem to find a route between locations on a `Map`.
    Create a problem with RouteProblem(start, goal, map=Map(...)}).
    States are the vertexes in the Map graph; actions are destination states."""
    
    def actions(self, state): 
        """The places neighboring `state`."""
        return self.map.neighbors[state]
    
    def result(self, state, action):
        """Go to the `action` place, if the map says that is possible."""
        return action if action in self.map.neighbors[state] else state
    
    def action_cost(self, s, action, s1):
        """The distance (cost) to go from s to s1."""
        return self.map.distances[s, s1]
    
    def h(self, node):
        "Straight-line distance between state and the goal."
        locs = self.map.locations
        return straight_line_distance(locs[node.state], locs[self.goal])
class Map:
    """A map of places in a 2D world: a graph with vertexes and links between them. 
    In `Map(links, locations)`, `links` can be either [(v1, v2)...] pairs, 
    or a {(v1, v2): distance...} dict. Optional `locations` can be {v1: (x, y)} 
    If `directed=False` then for every (v1, v2) link, we add a (v2, v1) link."""

    def __init__(self, links, locations=None, directed=False):
        if not hasattr(links, 'items'): # Distances are 1 by default
            links = {link: 1 for link in links}
        if not directed:
            for (v1, v2) in list(links):
                links[v2, v1] = links[v1, v2]
        self.distances = links
        self.neighbors = multimap(links)
        self.locations = locations or defaultdict(lambda: (0, 0))

        
def multimap(pairs) -> dict:
    "Given (key, val) pairs, make a dict of {key: [val,...]}."
    result = defaultdict(list)
    for key, val in pairs:
        result[key].append(val)
    return result

Home_nearby_locations = Map(
    {('Kundrathur', 'Home'): 6, ('Kundrathur', 'Pammal'): 6,
     ('Home', 'Porur'): 4, ('Pammal', 'Airport'):5,
     ('Porur', 'Vadapalani'): 7, ('Porur', 'Maduravoyal'): 4, ('Porur', 'Guindy'): 10, ('Airport', 'Guindy'): 9,
     ('Vadapalani', 'T.Nagar'): 4, ('Vadapalani', 'Koyambedu'): 4, ('Maduravoyal', 'Koyambedu'): 5, ('Maduravoyal', 'Ambattur'): 6, ('Guindy', 'Saidapet'): 2,
     ('T.Nagar', 'EA Mall'): 5, ('Koyambedu', 'Korattur'): 6, ('Ambattur', 'Madhavaram'): 13, ('Saidapet', 'T.Nagar'): 4,
     ('EA Mall', 'Broadway'): 5, ('Korattur', 'Mahavaram'): 10, ('Madhavaram', 'Manali'): 11,
     ('Broadway', 'Tondiarpet'): 5, ('Broadway', 'Perambur'): 9, ('Manali', 'Tiruvottiyur'): 7,
     ('Tondiarpet', 'Tiruvottiyur'): 6, ('Perambur', 'Madhavaram'): 5})


r0 = RouteProblem('Home', 'Tiruvottiyur', map=Home_nearby_locations)
r1 = RouteProblem('Vadapalani', 'Tiruvottiyur', map=Home_nearby_locations)
r2 = RouteProblem('Guindy', 'Madhavaram', map=Home_nearby_locations)
r3 = RouteProblem('Kundrathur', 'Manali', map=Home_nearby_locations)
r4 = RouteProblem('Perambur', 'Home', map=Home_nearby_locations)


goal_state_path_0=best_first_search(r0,g)
goal_state_path_1=best_first_search(r1,g)
goal_state_path_2=best_first_search(r2,g)
goal_state_path_3=best_first_search(r3,g)
goal_state_path_4=best_first_search(r4,g)
print("GoalStateWithPath:{0}".format(goal_state_path_0))
print("Total Distance={0} Kilometers".format(goal_state_path_0.path_cost))
print("Route:{0}".format(path_states(goal_state_path_0)))

print("\nGoalStateWithPath:{0}".format(goal_state_path_1))
print("Total Distance={0} Kilometers".format(goal_state_path_1.path_cost))
print("Route:{0}".format(path_states(goal_state_path_1)))

print("\nGoalStateWithPath:{0}".format(goal_state_path_2))
print("Total Distance={0} Kilometers".format(goal_state_path_2.path_cost))
print("Route:{0}".format(path_states(goal_state_path_2)))
print("\nGoalStateWithPath:{0}".format(goal_state_path_3))
print("Total Distance={0} Kilometers".format(goal_state_path_3.path_cost))
print("Route:{0}".format(path_states(goal_state_path_3)))
print("\nGoalStateWithPath:{0}".format(goal_state_path_4))
print("Total Distance={0} Kilometers".format(goal_state_path_4.path_cost))
print("Route:{0}".format(path_states(goal_state_path_4)))
```

## OUTPUT:
![Screenshot (188)](https://user-images.githubusercontent.com/75234807/167794722-656729cc-7f8c-430d-950f-7eaa22001558.png)

## Solution Justification:
In best-first search algorithm, the selected node is verified as parent node or not and starts its search, within the least distance it will be reaching the goal node. Search near every two nodes are always considered with its shortest distance.

## RESULT:
Thus an algorithm to find the route from the source to the destination point using best-first search is developed and executed successfully.

