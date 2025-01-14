---
layout: exercise_python
permalink: "Module20/Exercise1"
title: "CS 371: Module 20: Exercise 1: Dijkstra Shortest Path Backtracing"
excerpt: "CS 371: Module 20: Exercise 1: Dijkstra Shortest Path Backtracing"
canvasasmtid: "117483"
canvaspoints: "1.5"
canvashalftries: 5

info:
  comments: "true"
  prev: "./Video2"
  points: 1.5
  instructions: "Study the code below, which implements a vanilla version of Dijkstra's algorithm and remembers the node that came before, <code>node.prev</code> when <code>node</code> first comes out of the queue.  Your job will be to fill in the method <code>get_path</code> to extract a shortest path from the starting node to a particular node using the <code>prev</code> references stored in the nodes after running Dijkstra's algorithm.  Most of this code is completed already, but you have to add on the current node and move to the previous node before the next iteration.  Overall, this is a lot like backtracing in dynamic time warping."
  goals:
    - Implement backtracing for shortest paths after running Dijkstra's algorithm
    
processor:  
  correctfeedback: "Correct!!" 
  incorrectfeedback: "Try again"
  submitformlink: false
  feedbackprocess: | 
    var pos = feedbackString.trim();
  correctcheck: |
        pos.includes("[0, 35, 36, 39, 1]")
  incorrectchecks:
    - incorrectcheck: |
        pos.includes("[]")
      feedback: "Try again.  It looks like you aren't adding anything to the path.  Be sure to use .append to add on the node, then move to the next node with node.prev"
files:
  - filename: "Heap"
    name: heap
    ismain: false
    isreadonly: true
    isvisible: true
    height: 600
    code: | 
            class HeapTree(object):
                def __init__(self):
                    self._arr = []

                def __len__(self):
                    return len(self._arr)

                def _children(self, i):
                    """
                    Return the indices of the children of the node at index i

                    Parameters
                    ----------
                    i: int
                        Index of the element
                    
                    Returns
                    -------
                    list of ints: Indices of children.  There are 0, 1, or 2
                    """
                    children = []
                    if 2*i+1 < len(self._arr):
                        children.append(2*i+1)
                    if 2*i+2 < len(self._arr):
                        children.append(2*i+2)
                    return children
                
                def _parent(self, i):
                    """
                    Return the index of the parent of this node

                    Parameters
                    ----------
                    i: int
                        Index of node
                    
                    Returns
                    -------
                    int: index of parent
                    """
                    return (i-1)//2
                
                def _swap(self, i, j):
                    """
                    Swap two nodes in the underlying array

                    Parameters
                    ----------
                    i: int
                        Index of first node
                    j: int
                        Index of second node
                    """
                    self._arr[i], self._arr[j] = self._arr[j], self._arr[i]

                def _upheap(self, i):
                    """
                    Recursively bubble a node up the heap if the node above
                    it is greater

                    Parameters
                    ----------
                    i: int
                        Index of node to bubble up
                    """
                    parent = self._parent(i)
                    if i > 0 and self._arr[i][0] < self._arr[parent][0]:
                        self._swap(i, parent)
                        self._upheap(parent)
                
                def _downheap(self, i):
                    """
                    Recursively bubble a node down the heap while it's greater 
                    than one of its children

                    Parameters
                    ----------
                    i: int
                        Index of node to bubble down
                    """
                    children = self._children(i)
                    if len(children) > 0:
                        child = children[0]
                        if len(children) > 1:
                            c2 = children[1]
                            if self._arr[c2][0] < self._arr[child][0]:
                                child = c2
                        if self._arr[child][0] < self._arr[i][0]:
                            self._swap(i, child)
                            self._downheap(child)

                def push(self, entry):
                    """
                    Put a new value onto the heap

                    Parameters
                    ----------
                    entry: tuple (float, ...)
                        A tuple whose first entry is the priority (the rest is ignored)
                    """
                    self._arr.append(entry)
                    self._upheap(len(self._arr)-1)
                
                def peek(self):
                    """
                    Return the value with the highest priority from the heap

                    Returns
                    -------
                    tuple (float, ...)
                    """
                    assert(len(self) > 0)
                    return self._arr[0]

                def pop(self):
                    """
                    Remove the value with the highest priority from the heap and
                    return it

                    Returns
                    -------
                    tuple (float, ...)
                    """
                    assert(len(self) > 0)
                    ret = self._arr[0]
                    # Move the last element to the root and bubble
                    # down if necessary
                    self._arr[0] = self._arr[-1]
                    self._arr.pop()
                    self._downheap(0)
                    return ret


  - filename: "Graph Code"
    name: graph
    ismain: false
    isreadonly: true
    isvisible: true
    code: | 
            class GraphNode:
                def __init__(self, idx=-1):
                    self.idx = idx
                    self.edges = [] # List of nodes that are adjacent
                    self.dists = [] # Parallel list of distances of the edges to these nodes
                    self.data = {}
                    self.visited = False

            def make_example_graph():
                edges = [(33, 1, 0.149),(44, 93, 0.147),(78, 60, 0.137),(81, 78, 0.144),(83, 98, 0.226),(47, 63, 0.0539),(6, 81, 0.0376),(41, 90, 0.122),(47, 77, 0.0123),(59, 26, 0.113),(98, 72, 0.205),(39, 86, 0.19),(61, 95, 0.0816),(31, 89, 0.194),(32, 50, 0.115),(80, 92, 0.187),(85, 57, 0.0465),(69, 75, 0.0773),(52, 72, 0.0393),(39, 1, 0.165),(15, 67, 0.0379),(1, 13, 0.212),(28, 88, 0.0884),(46, 69, 0.135),(65, 40, 0.0418),(59, 54, 0.0652),(16, 14, 0.0843),(48, 28, 0.213),(59, 71, 0.225),(76, 29, 0.143),(33, 96, 0.0731),(68, 23, 0.103),(90, 57, 0.183),(23, 17, 0.0815),(60, 14, 0.146),(37, 2, 0.0832),(1, 39, 0.165),(39, 33, 0.175),(89, 31, 0.194),(71, 59, 0.225),(35, 5, 0.0319),(78, 48, 0.0216),(28, 51, 0.224),(21, 17, 0.0867),(87, 82, 0.0324),(25, 35, 0.0272),(2, 62, 0.0552),(24, 15, 0.0414),(98, 1, 0.241),(68, 18, 0.163),(57, 87, 0.161),(19, 38, 0.0841),(30, 53, 0.163),(32, 96, 0.16),(96, 33, 0.0731),(92, 80, 0.187),(18, 20, 0.254),(74, 23, 0.155),(46, 26, 0.105),(70, 21, 0.178),(93, 18, 0.088),(20, 38, 0.081),(60, 53, 0.0997),(0, 88, 0.0589),(92, 69, 0.0386),(56, 62, 0.0853),(44, 74, 0.0736),(20, 18, 0.254),(53, 60, 0.0997),(11, 51, 0.0929),(11, 88, 0.125),(55, 47, 0.0785),(32, 4, 0.118),(96, 50, 0.0797),(95, 24, 0.093),(93, 84, 0.15),(87, 95, 0.134),(92, 75, 0.1),(72, 8, 0.0265),(47, 43, 0.074),(76, 90, 0.0638),(15, 24, 0.0414),(54, 46, 0.0236),(28, 81, 0.114),(12, 37, 0.0673),(5, 25, 0.0308),(6, 86, 0.103),(76, 95, 0.176),(35, 25, 0.0272),(7, 38, 0.0953),(6, 58, 0.187),(20, 68, 0.141),(90, 58, 0.148),(75, 97, 0.0359),(67, 75, 0.148),(61, 34, 0.148),(71, 80, 0.187),(43, 14, 0.0789),(21, 70, 0.178),(56, 11, 0.13),(39, 58, 0.217),(99, 15, 0.0907),(71, 26, 0.327),(49, 22, 0.107),(0, 28, 0.0322),(70, 10, 0.199),(14, 60, 0.146),(35, 36, 0.0175),(76, 24, 0.182),(70, 72, 0.321),(14, 43, 0.0789),(61, 87, 0.137),(95, 90, 0.164),(5, 35, 0.0319),(67, 97, 0.131),(21, 10, 0.0547),(37, 25, 0.0521),(94, 55, 0.168),(70, 38, 0.135),(42, 5, 0.15),(19, 7, 0.122),(57, 90, 0.183),(40, 64, 0.181),(89, 13, 0.015),(94, 78, 0.0358),(91, 83, 0.0376),(83, 96, 0.106),(84, 11, 0.0956),(90, 95, 0.164),(95, 76, 0.176),(28, 11, 0.186),(36, 28, 0.125),(74, 68, 0.117),(70, 20, 0.204),(16, 43, 0.0414),(11, 56, 0.13),(10, 89, 0.193),(60, 78, 0.137),(50, 71, 0.139),(79, 26, 0.0277),(63, 47, 0.0539),(40, 65, 0.0418),(29, 4, 0.0423),(83, 1, 0.0393),(2, 88, 0.0273),(96, 91, 0.0828),(54, 69, 0.122),(61, 82, 0.132),(5, 42, 0.15),(0, 35, 0.111),(21, 23, 0.144),(68, 74, 0.117),(81, 6, 0.0376),(47, 64, 0.0879),(62, 56, 0.0853),(71, 50, 0.139),(2, 37, 0.0832),(34, 16, 0.311),(2, 12, 0.0498),(91, 96, 0.0828),(51, 65, 0.071),(12, 2, 0.0498),(89, 70, 0.214),(75, 69, 0.0773),(53, 34, 0.0949),(87, 57, 0.161),(24, 76, 0.182),(87, 53, 0.0269),(43, 47, 0.074),(11, 22, 0.129),(51, 28, 0.224),(77, 47, 0.0123),(18, 74, 0.0962),(81, 28, 0.114),(48, 78, 0.0216),(16, 99, 0.447),(36, 35, 0.0175),(90, 41, 0.122),(8, 13, 0.0835),(26, 73, 0.461),(81, 48, 0.136),(77, 64, 0.0899),(57, 53, 0.158),(38, 19, 0.0841),(44, 56, 0.0137),(14, 34, 0.249),(36, 5, 0.0338),(29, 80, 0.158),(72, 70, 0.321),(73, 26, 0.461),(68, 19, 0.0493),(64, 77, 0.0899),(53, 82, 0.0539),(33, 41, 0.149),(66, 73, 0.216),(23, 19, 0.102),(58, 90, 0.148),(4, 96, 0.164),(73, 71, 0.141),(84, 3, 0.0354),(10, 31, 0.0254),(54, 92, 0.0992),(55, 63, 0.0295),(1, 83, 0.0393),(82, 61, 0.132),(42, 21, 0.121),(86, 28, 0.125),(23, 68, 0.103),(3, 49, 0.185),(83, 91, 0.0376),(40, 49, 0.102),(20, 70, 0.204),(81, 57, 0.169),(87, 61, 0.137),(59, 46, 0.0461),(93, 49, 0.357),(73, 66, 0.216),(85, 81, 0.149),(49, 9, 0.0501),(43, 77, 0.07),(39, 36, 0.159),(19, 68, 0.0493),(74, 62, 0.11),(76, 67, 0.186),(76, 92, 0.219),(13, 8, 0.0835),(14, 53, 0.201),(28, 48, 0.213),(78, 30, 0.092),(70, 27, 0.222),(69, 79, 0.142),(9, 49, 0.0501),(69, 92, 0.0386),(7, 70, 0.099),(84, 22, 0.107),(65, 48, 0.151),(43, 16, 0.0414),(80, 4, 0.135),(5, 36, 0.0338),(39, 5, 0.149),(32, 71, 0.0821),(67, 15, 0.0379),(66, 98, 0.047),(62, 37, 0.0793),(50, 91, 0.114),(30, 78, 0.092),(55, 94, 0.168),(62, 74, 0.11),(41, 58, 0.0493),(13, 1, 0.212),(50, 32, 0.115),(86, 6, 0.103),(29, 76, 0.143),(51, 48, 0.197),(54, 59, 0.0652),(58, 41, 0.0493),(82, 53, 0.0539),(63, 14, 0.0853),(29, 33, 0.154),(88, 12, 0.0316),(52, 98, 0.191),(26, 59, 0.113),(52, 70, 0.359),(89, 27, 0.0185),(10, 21, 0.0547),(28, 36, 0.125),(7, 21, 0.117),(51, 40, 0.084),(64, 40, 0.181),(89, 10, 0.193),(37, 0, 0.0724),(98, 73, 0.228),(53, 57, 0.158),(96, 4, 0.164),(98, 52, 0.191),(33, 29, 0.154),(97, 79, 0.243),(46, 54, 0.0236),(53, 87, 0.0269),(78, 94, 0.0358),(56, 93, 0.151),(31, 1, 0.169),(10, 70, 0.199),(12, 0, 0.029),(16, 34, 0.311),(28, 86, 0.125),(4, 80, 0.135),(36, 39, 0.159),(77, 16, 0.11),(88, 11, 0.125),(21, 5, 0.171),(59, 80, 0.121),(85, 30, 0.00413),(23, 74, 0.155),(39, 42, 0.0164),(5, 23, 0.151),(70, 52, 0.359),(90, 76, 0.0638),(27, 72, 0.107),(32, 80, 0.141),(57, 85, 0.0465),(98, 83, 0.226),(71, 73, 0.141),(30, 85, 0.00413),(24, 61, 0.0307),(90, 29, 0.145),(57, 81, 0.169),(64, 16, 0.2),(57, 30, 0.0502),(48, 51, 0.197),(67, 92, 0.177),(91, 50, 0.114),(83, 33, 0.138),(90, 6, 0.227),(86, 36, 0.0969),(7, 17, 0.0598),(98, 8, 0.213),(5, 39, 0.149),(98, 91, 0.222),(73, 91, 0.201),(64, 55, 0.132),(74, 18, 0.0962),(13, 27, 0.0195),(30, 57, 0.0502),(69, 46, 0.135),(19, 17, 0.117),(50, 96, 0.0797),(40, 51, 0.084),(46, 79, 0.114),(23, 45, 0.125),(27, 13, 0.0195),(14, 63, 0.0853),(25, 23, 0.146),(1, 31, 0.169),(62, 44, 0.0848),(61, 99, 0.107),(22, 3, 0.0836),(97, 75, 0.0359),(13, 31, 0.2),(50, 73, 0.135),(25, 45, 0.102),(15, 99, 0.0907),(42, 39, 0.0164),(99, 97, 0.181),(8, 1, 0.253),(45, 23, 0.125),(9, 51, 0.101),(91, 73, 0.201),(7, 19, 0.122),(75, 92, 0.1),(33, 83, 0.138),(56, 2, 0.088),(5, 21, 0.171),(58, 33, 0.149),(26, 46, 0.105),(25, 5, 0.0308),(2, 56, 0.088),(13, 89, 0.015),(22, 49, 0.107),(42, 31, 0.0769),(94, 65, 0.141),(73, 50, 0.135),(6, 28, 0.108),(67, 76, 0.186),(8, 27, 0.0808),(4, 29, 0.0423),(34, 99, 0.136),(79, 69, 0.142),(38, 20, 0.081),(9, 40, 0.0611),(91, 98, 0.222),(98, 66, 0.047),(55, 14, 0.0965),(93, 3, 0.172),(95, 57, 0.105),(24, 95, 0.093),(26, 66, 0.678),(67, 99, 0.119),(60, 30, 0.132),(38, 70, 0.135),(33, 4, 0.153),(38, 7, 0.0953),(12, 88, 0.0316),(16, 77, 0.11),(30, 81, 0.15),(45, 37, 0.0738),(8, 98, 0.213),(17, 21, 0.0867),(28, 6, 0.108),(53, 30, 0.163),(3, 93, 0.172),(88, 2, 0.0273),(58, 6, 0.187),(79, 75, 0.208),(72, 52, 0.0393),(40, 55, 0.204),(11, 84, 0.0956),(34, 53, 0.0949),(72, 98, 0.205),(49, 64, 0.169),(61, 15, 0.0616),(48, 81, 0.136),(42, 10, 0.0948),(23, 21, 0.144),(28, 35, 0.118),(80, 76, 0.184),(4, 33, 0.153),(74, 44, 0.0736),(17, 19, 0.117),(61, 24, 0.0307),(18, 44, 0.135),(44, 18, 0.135),(9, 22, 0.0781),(57, 6, 0.186),(99, 34, 0.136),(1, 33, 0.149),(26, 71, 0.327),(67, 24, 0.0673),(92, 76, 0.219),(42, 1, 0.167),(94, 60, 0.137),(34, 82, 0.0459),(27, 89, 0.0185),(65, 51, 0.071),(70, 7, 0.099),(95, 87, 0.134),(93, 44, 0.147),(24, 67, 0.0673),(41, 33, 0.149),(81, 85, 0.149),(15, 61, 0.0616),(31, 13, 0.2),(23, 25, 0.146),(44, 62, 0.0848),(14, 55, 0.0965),(51, 9, 0.101),(37, 35, 0.0623),(21, 7, 0.117),(6, 90, 0.227),(3, 84, 0.0354),(84, 56, 0.154),(2, 11, 0.134),(51, 11, 0.0929),(29, 41, 0.0806),(31, 10, 0.0254),(63, 43, 0.082),(17, 7, 0.0598),(22, 9, 0.0781),(75, 67, 0.148),(37, 45, 0.0738),(21, 42, 0.121),(34, 14, 0.249),(78, 81, 0.144),(48, 65, 0.151),(1, 8, 0.253),(80, 71, 0.187),(48, 94, 0.0363),(37, 12, 0.0673),(8, 72, 0.0265),(51, 22, 0.0909),(68, 20, 0.141),(30, 60, 0.132),(57, 95, 0.105),(22, 11, 0.129),(27, 70, 0.222),(22, 51, 0.0909),(19, 20, 0.113),(54, 80, 0.117),(63, 55, 0.0295),(19, 23, 0.102),(74, 45, 0.113),(94, 48, 0.0363),(88, 0, 0.0589),(55, 40, 0.204),(65, 55, 0.208),(84, 93, 0.15),(86, 58, 0.141),(35, 28, 0.118),(35, 37, 0.0623),(40, 9, 0.0611),(69, 54, 0.122),(27, 8, 0.0808),(92, 67, 0.177),(49, 3, 0.185),(43, 63, 0.082),(4, 32, 0.118),(36, 86, 0.0969),(58, 86, 0.141),(17, 23, 0.0815),(6, 57, 0.186),(95, 61, 0.0816),(60, 94, 0.137),(53, 14, 0.201),(62, 2, 0.0552),(58, 39, 0.217),(20, 93, 0.342),(3, 22, 0.0836),(64, 47, 0.0879),(65, 94, 0.141),(70, 89, 0.214),(55, 60, 0.151),(75, 79, 0.208),(55, 64, 0.132),(11, 2, 0.134),(1, 98, 0.241),(81, 30, 0.15),(82, 34, 0.0459),(1, 42, 0.167),(45, 25, 0.102),(55, 65, 0.208),(66, 52, 0.225),(33, 58, 0.149),(23, 5, 0.151),(31, 42, 0.0769),(96, 83, 0.106),(99, 67, 0.119),(47, 55, 0.0785),(93, 56, 0.151),(88, 28, 0.0884),(34, 61, 0.148),(71, 32, 0.0821),(56, 84, 0.154),(72, 27, 0.107),(52, 20, 0.563),(29, 90, 0.145),(92, 54, 0.0992),(18, 68, 0.163),(10, 42, 0.0948),(49, 40, 0.102),(80, 32, 0.141),(77, 43, 0.07),(45, 62, 0.0231),(22, 84, 0.107),(33, 39, 0.175),(18, 93, 0.088),(80, 59, 0.121),(73, 98, 0.228),(0, 37, 0.0724),(82, 87, 0.0324),(80, 54, 0.117),(25, 37, 0.0521),(0, 12, 0.029),(79, 46, 0.114),(46, 59, 0.0461),(62, 45, 0.0231),(45, 74, 0.113),(28, 0, 0.0322),(76, 80, 0.184),(35, 0, 0.111),(96, 32, 0.16),(86, 39, 0.19),(37, 62, 0.0793),(80, 29, 0.158),(56, 44, 0.0137),(99, 61, 0.107),(97, 67, 0.131),(60, 55, 0.151),(14, 16, 0.0843),(20, 19, 0.113),(41, 29, 0.0806),(11, 28, 0.186)]
                nodes = []
                N = 0
                for (i, j, d) in edges:
                    N = max(N, i+1, j+1)
                nodes = [GraphNode(i) for i in range(N)]
                for (i, j, d) in edges:
                    nodes[i].edges.append(nodes[j])
                    nodes[i].dists.append(d)
                    nodes[j].edges.append(nodes[i])
                    nodes[j].dists.append(d)
                for idx in range(len(edges)):
                    (i, j, d) = edges[idx]
                    edges[idx] = (nodes[i], nodes[j], d)
                return nodes, edges

  - filename: "Dijkstra"
    name: dijkstra
    ismain: false
    isreadonly: true
    isvisible: true
    height: 360
    code: | 
            def do_dijkstra(nodes, edges):
                for n in nodes:
                    n.visited = False
                queue = HeapTree()
                nodes[0].prev = None
                queue.push((0, nodes[0], None))
                while len(queue) > 0:
                    (d, n, nprev) = queue.pop()
                    if not n.visited:
                        n.visited = True
                        n.prev = nprev # Remember the previous node
                        for i in range(len(n.edges)):
                            ni = n.edges[i]
                            if not ni.visited:
                                ni.touched = True
                                di = n.dists[i]
                                queue.push((d+di, ni, n))

  - filename: "Student code"
    name: student
    ismain: false
    isreadonly: false
    isvisible: true
    height: 600
    code: | 
            def get_path(start_node):
                """
                Trace a path back from a particular node
                to the starting node

                Parameters
                ----------
                node: GraphNode
                    The starting node
                
                Returns
                -------
                list of int: indices of the path from the
                starting node to this node
                """
                max_iters = 1000 # Safeguard for infinite loop
                node = start_node
                path = []
                iter = 0
                while node and iter < max_iters:
                    ## TODO: Fill this in
                    
                    iter += 1
                path.reverse()
                path = [n.idx for n in path]
                return path

  - filename: "Test Code Block"
    ismain: true
    name: main
    isreadonly: true
    isvisible: true
    code: |
            nodes, edges = make_example_graph()
            do_dijkstra(nodes, edges)
            print(get_path(nodes[1]))

---
