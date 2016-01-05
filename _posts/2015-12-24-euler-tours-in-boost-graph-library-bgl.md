---
layout: post
title: Euler Tours in Boost Graph Library (BGL)
category: projects
---

Just looking for algorithm sources? [Source code](/files/eulerian_circuit_algorithms.zip "Eulerian Circuit Algorithms")

### Overview
A Eulerian tour is a path along edges of a graph that visits every edge exactly once. If the path also starts and begins at the same vertex it is called an Eulerian circuit. This set of algorithms provides utilities for finding the existence of Eulerian paths and circuits and also provides algorithms to compute them. Here is an example of an Eulerian circuit:

<p class="text-center">
  <img src="/images/graph-with-euler-circuit.png" alt="A graph with an Eulerian Circuit"> <br>
  This graph has Eulerian Cycles, for example: <br>
  B &rarr; E &rarr; C &rarr; B &rarr; A &rarr; D &rarr; B
</p>


The image above shows one example of a cycle that starts at vertex number 2 but it isn't a unique solution. Eulerian cycles can in fact start at any vertex of the graph and return to the same vertex at the end once all edges have been traversed. This is a useful property of Eulerian tours/circuits: every edge is traversed.

Eulerian trails have many real world uses including those in bioinformatics and circuit design. Euler tours can also be used more generally for providing a framework of visiting all edges in a more useful way than simply iterating through them randomly.

The constraints for a euler tour/circuit to exist are quite simple. For a cycle/circuit to exist, each vertex in an undirected graph must have an even number of incident edges and all vertices must belong to a single connected component. For a path to exist, at most two vertices can have an odd degree and all vertices must belong to a single connected component. Note that the path is different than the circuit.

These utilities are provided to check for the existence of and compute Eulerian paths and cycles for undirected graphs represented by Boost Graph Library data structures.

### Examples
See `example.cpp` for example source code. On a system with boost installed, the included `Makefile` should compile it into a binary called `example.bin` on linux systems with the command `make example`. Then, simply run with the command `./example.bin`.

Additionally, an undirected graph can be defined in a `txt` file in an adjacency list format and be read in by `euler_tour_tester`. The format is as follows:

<table class="table">
    <tbody>
        <tr>
            <th>
                Format
            </th>
            <th>
                Example (a ring graph)
            </th>
        </tr>
        <tr>
            <td>
                &lt;number of vertices&gt; <br>
                &lt;undirected edge between two vertices&gt; <br>
                &lt;undirected edge between two vertices&gt; <br>
                &lt;undirected edge between two vertices&gt; <br>
            </td>
            <td>
                3 <br>
                0 1 <br>
                1 2 <br>
                2 0 <br>
            </td>
        </tr>
    </tbody>
</table>

Once a file is created, it can be fed into the program by compiling the program with `make euler_tour_tester` and then running `./euler_tour_tester.bin <path_to_file.txt>`. It's useful to note there are a couple of debugging `#define` statements at the top of `euler_tour_tester.cpp` that will print extra info.

### Usage
Typical usage of the library involved retrieving a euler tour from a graph and then doing something with it. Since the tour is computed and inserted into a vector, there are endless possible directions one might go. For example, one could write a program similar to `euler_tour_tester.cpp` that reads in a file and then passes it into the algorithm and then print out the path. Or, one could modify it to read from standard input and then one could read and write files with unix I/O redirections operators `>` and `<`.

A typical usage program like the one described above can be found in `usage_example.cpp`. To use it simply compile it with `make usage_example.cpp` and then run like: `./usage_example.bin < input_graph.txt > output_tour.txt`.

## Reference manual
The functions implemented with this library provide ways for checking BGL undirected graphs for the existence of Euler tours as well as computing an Eulerian path for undirected graphs. In general, a BGL graph must be a single connected component and have all even degrees to have an Eulerian tour. If either of these invariants are broken, there is no path that can be formed.

#### `has_euler_(path|circuit)`
Both of these functions do a simple DFS on the entire graph to check that it is one entire connected component and that all vertices are even for the existence of a circuit or that at most two vertices are odd for the existence of a path. If the conditions for the existence of a path/circuit are met, the functions will return `true` otherwise `false`. The signatures are below:

{% highlight cpp %}
template <typename Graph>
bool has_euler_path(const Graph& G)

template <typename Graph>
bool has_euler_circuit(const Graph& G)
{% endhighlight %}

The parameter Graph must be a graph that can be used in the standard Boost DFS search.

#### `compute_euler`

{% highlight cpp %}
template <typename Graph, typename Edge = typename graph_traits<Graph>::edge_descriptor>
void compute_euler(const Graph& g, std::vector<Edge>& path, bool compute_circuit)
{% endhighlight %}

The `compute_euler()` function performs the computation of an eulerian path or circuit for the passed in graph and adds the edges in path/circuit order to the path vector. The function assumes that the existence of a path or circuit in the graph has already been proven true by `has_euler_path()` or `has_euler_circuit()`. The function chooses a starting vertex and then computes the path using Hierholzer's algorithm.

This algorithm uses two data structures to operate: a stack to push vertices to as they are visited and a map to color edges as they are traversed. Edges begin white to represent unvisited and then are colored grey once they have been used. The algorithm starts at a vertex and then starts touring the graph in a DFS style traversal keeping track of vertices in the stack and then pops them off when they have run out of unused incident edges. Pseudo-code for the algorithm is listed below.


    compute_euler(G, path, is_circuit)
     s := V[g][0]                        initialize the start vertex
     if (is_circuit)
      for each vertex u in V[G]
       if (is_odd(u))
        s := u                           choose an odd vertex if computing a path
        break
       end if
      end for
     end if
     for each edge e in E[G]             initialize edge colors
      color[e] := WHITE
     end for
     tmp_vertex := null
     PUSH(s, stack)                      push start vertex to stack
     while (SIZE(stack) != 0)
      u := TOP(stack)
      edge_available := false
      for each edge e in out_edges[u]    search for an unused incident edge
       if (color[e] == WHITE)
        edge_available := true
        color[e] := BLACK
        neighbor := target(e)
        PUSH(neighbor, stack)            push the edge's target vertex to the stack
        break
       end if
      end for
      if (NOT(edge_available)            if edge was unavailable, backtrack and add
       if (tmp_vertex != null)           to final path
        edge := edge(tmp_vertex, u, G)
        ADD(edge, path)
       end if
       tmp_vertex := u
       POP(stack)
     end if
     end while
    end

The two other computation functions `compute_euler_path()` and `compute_euler_circuit()` are aliases to `compute_euler()` above but also first check for the existence of a path or circuit first respectively. If a path or circuit doesn't exist, the functions will return early and the path vector will remain empty. The signatures for these functions are:

{% highlight cpp %}
template <typename Graph, typename Edge = typename graph_traits<Graph>::edge_descriptor>
void compute_euler_path(const Graph& g, std::vector<Edge>& path)

template <typename Graph, typename Edge = typename graph_traits<Graph>::edge_descriptor>
void compute_euler_circuit(const Graph& g, std::vector<Edge>& path)
{% endhighlight %}

The parameters are as follows:

- `Graph g`: A graph that supports the following BGL features:
  - edge(u, v, g)
  - target(e, g)
  - source(e, g)
  - vertices(g)
  - out_edges(g)
- `std::vector<Edge> path`: a vector that contains Edges of type `graph_traits<Graph>::edge_descriptor`

### Usage
The following tests were performed on an Ubuntu 14.04 system with g++ version 4.8.4. The makefile includes the exact compilation commands that were used to compile the source and the testing code.

- Compile the testing utility with `make euler_tour_tester`
- Choose one of the sample graph files or create your own (instructions above).
- Run the utility with the graph test file like: `./euler_tour_tester.bin sample_graphs/ring_graph.txt`
- Or, just run `./euler_tour_tester.bin` and then enter a number when prompted to create a large random eulerian circuit graph.

## Source code

Sources listed below are [available here](/files/eulerian_circuit_algorithms.zip "Eulerian Circuit Algorithms").

### Algorithms

{% highlight cpp %}
/*
 * Algorithms to test for the existence of and compute
 * Eulerian tours.
 *
 * Author: Brent Walther
 * Last Updated: Dec 27, 2015
 */

#include <boost/graph/adjacency_list.hpp>
#include <boost/graph/depth_first_search.hpp>
#include <boost/graph/undirected_dfs.hpp>

using namespace boost;

class default_euler_tour_visitor {
public:
  virtual double tour_edge() = 0;
private:
  default_euler_tour_visitor() { }
};

template <typename Counter>
class dfs_odd_even_visitor : public default_dfs_visitor {
 public:
  dfs_odd_even_visitor(Counter counts) : counts(counts) { }

  template <typename Vertex, typename Graph>
  void finish_vertex(Vertex u, const Graph& g) const {
    if (counts[0] > 2) {
       // more than one odd vertex means an euler tour is not possible. return early
      return;
    }
    typename graph_traits<Graph>::edges_size_type num_edges = 0;
    typename graph_traits<Graph>::adjacency_iterator vi, vi_end, next;
    for (tie(vi, vi_end) = adjacent_vertices(u, g); vi != vi_end; vi++) {
      num_edges++;
    }
    if (num_edges % 2 == 1) {
      counts[0]++;
    }
  }

  template <typename Vertex, typename Graph>
  void start_vertex(Vertex u, const Graph& g) {
    if (counts[1] > 1) {
      // more than one disconnected component means an euler tour is impossible. return early
      return;
    }
    counts[1]++;
  }

 private:
  Counter counts;
};

template <typename Graph>
std::vector<int>
get_graph_counts(const Graph& G) {
  std::vector<int> counts(2);
  dfs_odd_even_visitor<int*> vis(&counts[0]);
  try {
    depth_first_search(G, visitor(vis));
  } catch (...) { /* early return */ }
  return counts;
}

template <typename Graph, typename Edge = typename graph_traits<Graph>::edge_descriptor>
void
compute_euler(const Graph& g, std::vector<Edge>& path, bool compute_circuit) {

  typedef typename graph_traits<Graph>::vertex_descriptor Vertex;
  typedef typename graph_traits<Graph>::vertex_iterator VertexIterator;

  Vertex start_vertex;
  VertexIterator vi, vi_end;
  //printf("coloring all nodes white\n");
  for (tie(vi, vi_end) = vertices(g), start_vertex = *vi; vi != vi_end; ++vi) {
    if (!compute_circuit) {
      typename graph_traits<Graph>::out_edge_iterator ei, ei_end;
      int num_adjacent = 0;
      for (tie(ei, ei_end) = out_edges(*vi, g); ei != ei_end; ++ei) {
        num_adjacent++;
      }
      if (num_adjacent % 2 == 1) {
        start_vertex = *vi;
      }
    }
  }

  std::set<Edge> visited_edges;
  std::vector<Vertex> tmp;
  std::vector<Vertex> order;
  tmp.push_back(start_vertex);
  while (!tmp.empty()) {
    Vertex v = tmp.back();
    typename graph_traits<Graph>::out_edge_iterator edgei, edgei_end;
    bool has_available_edge = false;
    for (tie(edgei, edgei_end) = out_edges(v, g); edgei != edgei_end; ++edgei) {
      if (visited_edges.find(*edgei) == visited_edges.end()) { //unmarked
        has_available_edge = true;
        Edge e = *edgei;
        Edge opposite_edge = edge(target(e, g), source(e, g), g).first;
        visited_edges.insert(e);
        visited_edges.insert(opposite_edge);

        Vertex neighbor = target(*edgei, g);
        tmp.push_back(neighbor);
        break;
      }
    }
    if (!has_available_edge) {
      if (!order.empty()) {
        Edge e;
        bool exists;
        tie(e, exists) = edge(order.back(), v, g);
        if (!exists) { throw "Error! Tried to use an edge that doesn't exist!"; }
        path.push_back(e);
        order.pop_back();
      }
      order.push_back(v);
      tmp.pop_back();
    }
  }
}

template <typename Graph>
bool
has_euler_path(const Graph& G) {
  std::vector<int> counts = get_graph_counts(G);
  return counts[1] == 1 && (counts[0] == 0 || counts[0] == 2);
}

template <typename Graph>
bool
has_euler_circuit(const Graph& G) {
  std::vector<int> counts = get_graph_counts(G);
  return counts[1] == 1 && counts[0] == 0;
}

template <typename Graph, typename Edge = typename graph_traits<Graph>::edge_descriptor>
void
compute_euler_circuit(const Graph& g, std::vector<Edge>& path) {
  if (has_euler_circuit(g)) {
    compute_euler(g, path, true);
  }
}

template <typename Graph, typename Edge = typename graph_traits<Graph>::edge_descriptor>
void
compute_euler_path(const Graph& g, std::vector<Edge>& path) {
  if (has_euler_path(g)) {
    compute_euler(g, path, false);
  }
}
{% endhighlight %}

### Test code
The code below will read in a graph file and test the algorithms above.

{% highlight cpp %}
/*
 * Code to read in a graph or generate a random eulerian
 * graph and then test the validity of the algorithms.
 *
 * Author: Brent Walther
 * Last Updated: Dec 27, 2015
 */

#include <iostream>
#include <fstream>
#include <string>

#include <boost/tokenizer.hpp>
#include <sys/time.h>

#include "euler_tour.cpp"

#define PRINT_GRAPH_EDGES false
#define PRINT_PATHS false

using namespace boost;

int main(int argc, char* argv[]) {
  int has_input_file = true;
  if (argc != 2) {
    std::cerr << "No filename to read in. Creating a random graph instead.\n";
    has_input_file = false;
  }

  typedef adjacency_list <vecS, vecS, undirectedS> UndirectedGraph;
  typedef graph_traits<UndirectedGraph>::vertex_descriptor Vertex;
  typedef graph_traits<UndirectedGraph>::vertex_iterator VertexIterator;
  typedef graph_traits<UndirectedGraph>::edge_descriptor GraphEdge;
  typedef property_map<UndirectedGraph, vertex_index_t>::type IndexMap;

  // construct graph and print out the list of edges
  UndirectedGraph g;

  typedef std::pair<int, int> Edge;
  std::vector<Edge> edgeVec;
  int num_nodes = -1;

  if (has_input_file) {
    std::ifstream myfile (argv[1]);
    if (!myfile.is_open()) {
      std::cerr << "Could not open file.\n";
      has_input_file = false;
    }

    int count = 0;

    std::string line;
    while (myfile.is_open() && getline(myfile,line)) {
      char_separator<char> sep(" ");
      tokenizer<char_separator<char> > tokens(line, sep);
      if (count == 0) {
        int num_nodes = std::stoi(line);
        if (PRINT_GRAPH_EDGES) {
          std::cout << "Building a graph with " << num_nodes << " vertices.\n";
        }
      } else {
        int e1 = -1, e2 = -1;
        for (const auto& token : tokens) {
          int val = std::stoi(token);
          if (e1 == -1) {
            e1 = val;
          } else {
            e2 = val;
          }
        }
        add_edge(e1, e2, g);
        if (PRINT_GRAPH_EDGES) {
          std::cout << "Adding edge " << e1 << " -> " << e2 << "\n";
        }
      }
      count++;
    }
    myfile.close();
  }
  /* else = no input file */
  else {
    int tmp;
    std::cout << "Specify the number of nodes in the random graph: ";
    std::cin >> tmp;
    srand(time(NULL));
    num_nodes = tmp;
    int num_edges = num_nodes * 4;

    for (int i = 0; i < num_edges; i++) {
      if (i < num_nodes) {
        int a = -1;
        do {
          a = rand() % num_nodes;
        } while (a == i);
        add_edge(i, a, g);
      } else {
        int a = rand() % num_nodes;;
        int b = -1;
        do {
          b = rand() % num_nodes;
        } while (a == b);
        if (edge(a, b, g).second == false && edge(b, a, g).second == false) {
          add_edge(a, b, g);
        }
      }
    }

    std::vector<Vertex> odd_vertices;
    VertexIterator vi, vi_end;
    for (tie(vi, vi_end) = vertices(g); vi != vi_end; ++vi) {
      typename graph_traits<UndirectedGraph>::out_edge_iterator ei, ei_end;
      int num_adjacent = 0;
      for (tie(ei, ei_end) = out_edges(*vi, g); ei != ei_end; ++ei) {
        num_adjacent++;
      }
      if (num_adjacent % 2 == 1) {
        odd_vertices.push_back(*vi);
      }
    }
    if (PRINT_GRAPH_EDGES) {
      std::cout << "num odd vertices that need a fix: " << odd_vertices.size() << std::endl;
    }
    std::vector<Vertex>::iterator v;
    int num = num_vertices(g);
    for (v = odd_vertices.begin(); v != odd_vertices.end(); v++) {
      add_edge(*v, num, g);
    }
  }

  IndexMap index = get(vertex_index, g);
  if (PRINT_GRAPH_EDGES) {
    std::cout << "edges(g) = ";
    graph_traits<UndirectedGraph>::edge_iterator ei, ei_end;
    for (boost::tie(ei, ei_end) = edges(g); ei != ei_end; ++ei) {
        std::cout << "[" << index[source(*ei, g)]
                  << "," << index[target(*ei, g)] << "], ";
    }
    std::cout << std::endl;
  }

  // check for the existence of both a euler tour and circuit
  struct timeval start, end;
  gettimeofday(&start, NULL);
  bool has_path = has_euler_path<UndirectedGraph>(g);
  gettimeofday(&end, NULL);
  long long elapsed_time =   (end.tv_sec * (unsigned int)1e6 +   end.tv_usec) -
                   (start.tv_sec * (unsigned int)1e6 + start.tv_usec);
  std::cout << "has euler path? " << (has_path ? "yes" : "no");
  std::cout << " in " << elapsed_time << " microseconds." << std::endl;
  gettimeofday(&start, NULL);
  bool has_circuit = has_euler_circuit<UndirectedGraph>(g);
  gettimeofday(&end, NULL);
  elapsed_time =   (end.tv_sec * (unsigned int)1e6 +   end.tv_usec) -
                   (start.tv_sec * (unsigned int)1e6 + start.tv_usec);
  std::cout << "has euler circuit? " << (has_circuit ? "yes" : "no");
  std::cout << " in " << elapsed_time << " microseconds." << std::endl;

  std::vector<GraphEdge> euler_path;
  gettimeofday(&start, NULL);
  compute_euler_path<UndirectedGraph>(g, euler_path);
  gettimeofday(&end, NULL);
  elapsed_time =   (end.tv_sec * (unsigned int)1e6 +   end.tv_usec) -
                   (start.tv_sec * (unsigned int)1e6 + start.tv_usec);
  std::cout << "euler path computation took " << elapsed_time << " microseconds\n";

  bool path_okay = euler_path.size() == num_edges(g);
  for (auto ei = euler_path.begin(); ei != euler_path.end(); ei++) {
    if (PRINT_PATHS) {
      std::cout << "(" << index[source(*ei, g)]
                      << "," << index[target(*ei, g)] << ") ";
    }
    if ((ei + 1) != euler_path.end()) {
      path_okay &= (target(*ei, g) == source(*(ei + 1), g));
    }
  }
  std::cout << "path okay? " << (path_okay ? "yes" : "no") << std::endl;

  std::vector<GraphEdge> euler_circuit;
  gettimeofday(&start, NULL);
  compute_euler_circuit<UndirectedGraph>(g, euler_circuit);
  gettimeofday(&end, NULL);
  elapsed_time =   (end.tv_sec * (unsigned int)1e6 +   end.tv_usec) -
                   (start.tv_sec * (unsigned int)1e6 + start.tv_usec);
  std::cout << "euler circuit computation took " << elapsed_time << " microseconds\n";

  bool circuit_okay = euler_circuit.size() == num_edges(g);
  for (auto ei = euler_circuit.begin(); ei != euler_circuit.end(); ei++) {
    if (PRINT_PATHS) {
      std::cout << "(" << index[source(*ei, g)]
                      << "," << index[target(*ei, g)] << ") ";
    }
    if ((ei + 1) == euler_circuit.end()) {
      circuit_okay &= (target(*ei, g) == source(*euler_circuit.begin(), g));
    } else {
      circuit_okay &= (target(*ei, g) == source(*(ei + 1), g));
    }
  }
  std::cout << "circuit okay? " << (circuit_okay ? "yes" : "no") << std::endl;

  bool path_passed = (has_path && path_okay);
  bool circuit_passed = (has_circuit && circuit_okay);
  std::cout << "path passed? " << (path_passed ? "yes." : "no.") << std::endl;
  std::cout << "circuit passed? " << (circuit_passed ? "yes." : "no.") << std::endl;

  return 0;
}
{% endhighlight %}

### Usage Example
An example of how the algorithms can be used is shown below:

{% highlight cpp %}
/*
 * An example of using the eulerian tour algorithms.
 *
 * Author: Brent Walther
 * Last Updated: Dec 27, 2015
 */

#include <iostream>
#include <fstream>
#include <string>

#include <boost/tokenizer.hpp>
#include <sys/time.h>

#include "euler_tour.cpp"

#define PRINT_DEBUG_STATEMENTS false

using namespace boost;

int main(int argc, char* argv[]) {
  typedef adjacency_list <vecS, vecS, undirectedS> UndirectedGraph;
  typedef graph_traits<UndirectedGraph>::vertex_descriptor Vertex;
  typedef graph_traits<UndirectedGraph>::vertex_iterator VertexIterator;
  typedef graph_traits<UndirectedGraph>::edge_descriptor GraphEdge;
  typedef property_map<UndirectedGraph, vertex_index_t>::type IndexMap;

  // construct graph and print out the list of edges
  UndirectedGraph g;
  int num_nodes = -1, count = 0;

  std::string line;
  while (getline(std::cin, line)) {
    char_separator<char> sep(" ");
    tokenizer<char_separator<char> > tokens(line, sep);
    if (count == 0) {
      int num_nodes = std::stoi(line);
      if (PRINT_DEBUG_STATEMENTS) {
        std::cout << "Building a graph with " << num_nodes << " vertices.\n";
      }
    } else {
      int e1 = -1, e2 = -1;
      for (const auto& token : tokens) {
        int val = std::stoi(token);
        if (e1 == -1) {
          e1 = val;
        } else {
          e2 = val;
        }
      }
      add_edge(e1, e2, g);
      if (PRINT_DEBUG_STATEMENTS) {
        std::cout << "Adding edge " << e1 << " -> " << e2 << "\n";
      }
    }
    count++;
  }

  IndexMap index = get(vertex_index, g);
  std::vector<GraphEdge> euler_circuit;
  compute_euler_circuit<UndirectedGraph>(g, euler_circuit);

  for (auto ei = euler_circuit.begin(); ei != euler_circuit.end(); ei++) {
    std::cout << index[source(*ei, g)] << " " << index[target(*ei, g)] << std::endl;
  }
  std::cout << std::endl;

  return 0;
}
{% endhighlight %}

### Example Build Command

{% highlight sh %}
g++ -std=c++11 euler_tour.cpp euler_tour_tester.cpp -o euler_tour_tester.bin -lboost_system -lboost_graph
{% endhighlight %}

## Sources
- [http://iampandiyan.blogspot.com/2013/10/c-program-to-find-euler-path-or-euler.html](http://iampandiyan.blogspot.com/2013/10/c-program-to-find-euler-path-or-euler.html)
- [http://www.boost.org/doc/libs/1_59_0/libs/graph/doc/breadth_first_search.html](http://www.boost.org/doc/libs/1_59_0/libs/graph/doc/breadth_first_search.html)
- [https://en.wikipedia.org/wiki/Eulerian_path](https://en.wikipedia.org/wiki/Eulerian_path)
- [http://www.boost.org/doc/libs/1_59_0/libs/graph/doc/index.html](http://www.boost.org/doc/libs/1_59_0/libs/graph/doc/index.html)
- [http://www.boost.org/doc/libs/1_55_0/libs/graph/doc/graph_theory_review.html](http://www.boost.org/doc/libs/1_55_0/libs/graph/doc/graph_theory_review.html)
- [http://illuminations.nctm.org/Activity.aspx?id=3550](http://illuminations.nctm.org/Activity.aspx?id=3550)
