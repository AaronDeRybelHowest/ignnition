.. _generate-your-dataset:

Generate your dataset
=====================

Format of the dataset
---------------------

In order to properly feed the dataset to *IGNNITION*, the dataset must
be in *JSON* format. For this, the user must generate *JSON* files
--potentially many-- with the help of the well-known library *NetworkX*,
each of which can optionally be compressed into a *.tar.gz* file.
Furthermore, *IGNNITION* requires that the user manually separates the
training from the evaluation set into two different directories, the
paths of which must be specified in the *training_options.yaml* file in
their corresponding fields (check :ref:`train and evaluate <train-and-evaluate>`).

We would like to highlight that the dataset can contain potentially many
*JSON* files as well as many *tar.gz* files, each of which compressing
one single *JSON* file. The only restriction in this regard is that the
*JSON* files are valid and follow the scheme that we present below.

What should the dataset include?
--------------------------------

In order to generate the dataset to be fed to *IGNNITION*, the user must
generate the corresponding graphs with the help of the well-known
library *NetworkX*. Moreover, this library allows that, after each of
the corresponding graphs is created, it can easily be serialized as a
JSON file. When designing the different graphs, however, one must
remember that most of the fields require references of values, which
enables our model description to be agnostic to the actual
dataset. The reason is that in execution time, *IGNNITION* will gather
the corresponding values that each of these references points to.
Ultimately, this design principle, thus, requires users to make only
minor changes in the model to adapt it to a completely different
dataset.

This principle, however, imposes an important constraint that all the
references used in the model description file match the ones used in the
dataset. Below we provide a brief description of how a user can ensure
that this constraint is satisfied. Nevertheless, *IGNNITION*
incorporates an error-checking system (further explained in
:ref:`debugging assistant <debugging_assistant>`), which assists users in the
debugging of such aspects.

How to generate a sample?
-------------------------

We now review how we can generate a general sample which should give the user a good intuition to potentially build more complex examples.

Create the graph
~~~~~~~~~~~~~~~~

First of all, the user must create a general graph using the calls from
below.

.. code:: python

        import networkx as nx
        G = nx.DiGraph()

Notice, however, that the call presented below creates a directed graph,
but this needs not be the case. Alternatively, the user may define an
undirected graph as follows:

.. code:: python

        import networkx as nx
        G = nx.Graph()

Create the nodes
~~~~~~~~~~~~~~~~

Now we must populate this graph with the corresponding nodes. For this,
it is important to remember that we are considering a general case in
which we can have nodes of different types, and which thus, must be
treated differently. Consequently, each of the nodes must include a
field called *entity* which value is the name of the entity of the node.
For simplicity, let us consider a simple case with two entity types
*entity1* and *entity2*.

Apart from this field entity, the user must also include for each of the
nodes as many fields as features were defined in the model description
file. For instance, let us consider a case in which we define in the
model description file a single feature *f1* for nodes of *entity1* and
a feature named *f2* for nodes of *entity2*. Below we show how such
nodes could be created.

.. code:: python

         G.add_node('node1',
            entity='entity1',
            f1=v_1)

         G.add_node('node2',
            entity='entity2',
            f2=v_2)

Notice that the value assigned to each of the features might not be a
a single integer, as it could be an array of values which *IGNNITION* will
automatically identify and treat appropriately.

Create edges
~~~~~~~~~~~~

Now that all the nodes are created, we just need to create the edges
between these nodes. For simplicity, let us suppose we want to add an
edge between the two previous nodes *node1* and *node2*. In some cases,
moreover, we might want to include information regarding this edge which
can be later referenced in the model description file. E.g., in examples
of the field of chemistry, we might want to include a feature indicating
the type of bond that this edge represents. To do so, we follow the same idea as before, and we also include in the definition of the
edge the name of the parameter and its value.

.. code:: python

        G.add_edge('node1', 'node2', edge_param1= v_3)

Defining the label
~~~~~~~~~~~~~~~~~~

Finally, we just need to include the information on the label. In this
case, it is worth remembering that GNNs can work either in node label or
in graph label. The first will hence aim to make single predictions over
potentially every node of the graph, and the second over the whole
graph.

Node level
^^^^^^^^^^

In this type of problem, we must define a label for each of the nodes,
or at least for each of the nodes belonging to a certain entity type. To
do so, we just need to add a new parameter to each of the nodes that we
created before. To do so, we can simply add this parameter when first
created the node. Otherwise, we can do it as follows:

.. code:: python

        G.nodes['node1'][my_label_name] = l

Again, *l* may or may not be a single integer. Moreover, note that
*my\_label\_name* must match with the name of *output\_label* used in
the model\_description file.

Graph level
^^^^^^^^^^^

The second option is that we aim to make predictions over the whole
graph. In this case, we need to add this information, not for each of the
nodes but to the entire graph. To do so, again using the name used in
the model\_description file, we proceed as follows:

.. code:: python

        G.graph[my_label_name] = l

Serializing the graph
---------------------

Now that we have created a sample, we just need to serialize it to be
able to save it as a JSON file. For this, use the code from below:

.. code:: python

        from networkx.readwrite import json_graph
        training_data = []
        parsed_graph = json_graph.node_link_data(G)
        training_data.append(parsed_graph)

At this point, we might want to accumulate many of them before writing
the file using the *traning\_data* array. In any case, once we want to
write this information as a file, use the code from below:

.. code:: python

        import json
        with open('data.json', 'w') as json_file:
            json.dump(training_data, json_file)

Compress the file
-----------------

This is an optional step, but which we recommend since it can help to
considerably reduce the memory size of the dataset. This step consists
of compressing the file we just created to maintain a dataset of
compressed files. For this, use the following python instructions:

.. code:: python

        import tarfile
        tar = tarfile.open(path + "/sample_" + str(file_ctr) + ".tar.gz", "w:gz")
        tar.add('data.json')
        tar.close()
        os.remove('data.json')

Practical example
-----------------

So far we have covered how a general dataset can be generated.
Nevertheless, we are sure everything will be much more clear after
checking how a specific dataset is generated. To do so, take a look at
:ref:`quick tutorial <quick-step-by-step-tutorial>` where we cover in detail how to
create a dataset to solve the problem of the *Shortest-path*.
