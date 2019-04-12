
.. _examples:

Examples
========

The examples presented here are simply
`Jupyter <https://jupyter.org/>`__ session showing some simple LibLET
objects usage.

Type 1 grammars and production graphs
-------------------------------------

Let’s start defining a *monotonic* grammar for :math:`a^nb^nc^n`

.. code:: ipython3

    from liblet import Grammar
    
    G = Grammar.from_string("""
    S -> a b c
    S -> a S Q
    b Q c -> b b c c
    c Q -> Q c
    """, False)
    
    G




.. parsed-literal::

    Grammar(N={Q, S}, T={a, b, c}, P=(S -> a b c, S -> a S Q, b Q c -> b b c c, c Q -> Q c), S=S)



It can be convenient to show the productions as a table, with numbered
rows.

.. code:: ipython3

    from liblet import iter2table
    
    iter2table(G.P)




.. raw:: html

    <table><tr><th>0<td style="text-align:left"><pre>S -> a b c</pre>
    <tr><th>1<td style="text-align:left"><pre>S -> a S Q</pre>
    <tr><th>2<td style="text-align:left"><pre>b Q c -> b b c c</pre>
    <tr><th>3<td style="text-align:left"><pre>c Q -> Q c</pre></table>



It’s now time to create a *derivation* of :math:`a^2b^2c^2`

.. code:: ipython3

    from liblet import Derivation
    
    d = Derivation(G).step(1, 0).step(0, 1).step(3, 3).step(2, 2)
    d




.. parsed-literal::

    S -> a S Q -> a a b c Q -> a a b Q c -> a a b b c c



It can be quite illuminating to see the *production graph* for such
derivation

.. code:: ipython3

    from liblet import ProductionGraph
    
    ProductionGraph(d)




.. image:: examples_files/examples_8_0.svg



Context-free grammars and ambiguity
-----------------------------------

Assume we want to experiment with an ambiguous grammar and look for two
different leftmost derivation of the same sentence.

To this aim, let’s consider the following grammar and a short derivation
leading to and addition of three terminals

.. code:: ipython3

    G = Grammar.from_string("""
    E -> E + E
    E -> E * E
    E -> i
    """)
    
    d = Derivation(G).step(0, 0).step(0, 0)
    d




.. parsed-literal::

    E -> E + E -> E + E + E



What are the possible steps at this point? The ``possible_steps`` method
comes in handy, here is a (numbered) table of pairs :math:`(p, q)` where
:math:`p` is production number and :math:`q` the position of the
nonterminal that can be substituted:

.. code:: ipython3

    possible_steps = list(d.possible_steps())
    iter2table(possible_steps)




.. raw:: html

    <table><tr><th>0<td style="text-align:left"><pre>(0, 0)</pre>
    <tr><th>1<td style="text-align:left"><pre>(0, 2)</pre>
    <tr><th>2<td style="text-align:left"><pre>(0, 4)</pre>
    <tr><th>3<td style="text-align:left"><pre>(1, 0)</pre>
    <tr><th>4<td style="text-align:left"><pre>(1, 2)</pre>
    <tr><th>5<td style="text-align:left"><pre>(1, 4)</pre>
    <tr><th>6<td style="text-align:left"><pre>(2, 0)</pre>
    <tr><th>7<td style="text-align:left"><pre>(2, 2)</pre>
    <tr><th>8<td style="text-align:left"><pre>(2, 4)</pre></table>



If we look for just for leftmost derivations among the
:math:`(p, q)`\ s, we must keep just the :math:`p`\ s corresponding to
the :math:`q`\ s equal to the minimum of the possible :math:`q` values.
The following function can be used to such aim:

.. code:: ipython3

    from operator import itemgetter
    
    def filter_leftmost_prods(possible_steps):
        possible_steps = list(possible_steps)
        if possible_steps:
            min_q = min(possible_steps, key = itemgetter(1))[1]
            return map(itemgetter(0), filter(lambda ps: ps[1] == min_q, possible_steps))
        return tuple()
    
    list(filter_leftmost_prods(possible_steps))




.. parsed-literal::

    [0, 1, 2]



Now, using a ``Queue`` we can enumerate all the leftmost productions, we
can have a fancy generator that returns a new derivation each time
``next`` is called on it:

.. code:: ipython3

    from liblet import Queue
    
    def derivation_generator(G):
        Q = Queue([Derivation(G)])
        while Q:
            derivation = Q.dequeue()
            if set(derivation.sentential_form()) <= G.T: 
                yield derivation
            for nprod in filter_leftmost_prods(derivation.possible_steps()):
                Q.enqueue(derivation.leftmost(nprod))

Let’s collect the first 10 derivations

.. code:: ipython3

    derivation = derivation_generator(G)
    D = [next(derivation) for _ in range(10)]
    iter2table(D)




.. raw:: html

    <table><tr><th>0<td style="text-align:left"><pre>E -> i</pre>
    <tr><th>1<td style="text-align:left"><pre>E -> E + E -> i + E -> i + i</pre>
    <tr><th>2<td style="text-align:left"><pre>E -> E * E -> i * E -> i * i</pre>
    <tr><th>3<td style="text-align:left"><pre>E -> E + E -> E + E + E -> i + E + E -> i + i + E -> i + i + i</pre>
    <tr><th>4<td style="text-align:left"><pre>E -> E + E -> E * E + E -> i * E + E -> i * i + E -> i * i + i</pre>
    <tr><th>5<td style="text-align:left"><pre>E -> E + E -> i + E -> i + E + E -> i + i + E -> i + i + i</pre>
    <tr><th>6<td style="text-align:left"><pre>E -> E + E -> i + E -> i + E * E -> i + i * E -> i + i * i</pre>
    <tr><th>7<td style="text-align:left"><pre>E -> E * E -> E + E * E -> i + E * E -> i + i * E -> i + i * i</pre>
    <tr><th>8<td style="text-align:left"><pre>E -> E * E -> E * E * E -> i * E * E -> i * i * E -> i * i * i</pre>
    <tr><th>9<td style="text-align:left"><pre>E -> E * E -> i * E -> i * E + E -> i * i + E -> i * i + i</pre></table>



As one can easily see, derivations 6 and 7 produce the same sentence
``i + i * i`` but evidently with two different leftmost derivations. We
can give a look at the production graphs to better see what is
happening.

.. code:: ipython3

    from liblet import side_by_side
    
    side_by_side(ProductionGraph(D[6]), ProductionGraph(D[7]))




.. raw:: html

    <div><?xml version="1.0" encoding="UTF-8" standalone="no"?>
    <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
     "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
    <!-- Generated by graphviz version 2.40.1 (20161225.0304)
     -->
    <!-- Title: %3 Pages: 1 -->
    <svg width="243pt" height="230pt"
     viewBox="0.00 0.00 243.03 230.00" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
    <g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 226)">
    <title>%3</title>
    <polygon fill="#ffffff" stroke="transparent" points="-4,4 -4,-226 239.0347,-226 239.0347,4 -4,4"/>
    <!-- &#45;5493785780082770843 -->
    <!-- &#45;8751665616789673910 -->
    <!-- &#45;5493785780082770843&#45;&gt;&#45;8751665616789673910 -->
    <!-- &#45;5099837602000299771 -->
    <g id="node2" class="node">
    <title>&#45;5099837602000299771</title>
    <path fill="none" stroke="#000000" stroke-width=".25" d="M157.4447,-222C157.4447,-222 152.0729,-222 152.0729,-222 149.387,-222 146.7011,-219.3141 146.7011,-216.6282 146.7011,-216.6282 146.7011,-205.3718 146.7011,-205.3718 146.7011,-202.6859 149.387,-200 152.0729,-200 152.0729,-200 157.4447,-200 157.4447,-200 160.1306,-200 162.8165,-202.6859 162.8165,-205.3718 162.8165,-205.3718 162.8165,-216.6282 162.8165,-216.6282 162.8165,-219.3141 160.1306,-222 157.4447,-222"/>
    <text text-anchor="middle" x="154.7588" y="-206.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
    </g>
    <!-- &#45;5099836340803300948 -->
    <g id="node4" class="node">
    <title>&#45;5099836340803300948</title>
    <path fill="none" stroke="#000000" stroke-width=".25" d="M123.4447,-182C123.4447,-182 118.0729,-182 118.0729,-182 115.387,-182 112.7011,-179.3141 112.7011,-176.6282 112.7011,-176.6282 112.7011,-165.3718 112.7011,-165.3718 112.7011,-162.6859 115.387,-160 118.0729,-160 118.0729,-160 123.4447,-160 123.4447,-160 126.1306,-160 128.8165,-162.6859 128.8165,-165.3718 128.8165,-165.3718 128.8165,-176.6282 128.8165,-176.6282 128.8165,-179.3141 126.1306,-182 123.4447,-182"/>
    <text text-anchor="middle" x="120.7588" y="-166.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
    </g>
    <!-- &#45;5099837602000299771&#45;&gt;&#45;5099836340803300948 -->
    <g id="edge2" class="edge">
    <title>&#45;5099837602000299771&#45;&gt;&#45;5099836340803300948</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M146.5283,-201.3171C141.1827,-195.0281 134.2439,-186.8649 128.9134,-180.5937"/>
    </g>
    <!-- &#45;4827002429087198528 -->
    <g id="node5" class="node">
    <title>&#45;4827002429087198528</title>
    <path fill="none" stroke="#000000" stroke-width="1.25" d="M157.3907,-182C157.3907,-182 152.1268,-182 152.1268,-182 149.4949,-182 146.8629,-179.3681 146.8629,-176.7361 146.8629,-176.7361 146.8629,-165.2639 146.8629,-165.2639 146.8629,-162.6319 149.4949,-160 152.1268,-160 152.1268,-160 157.3907,-160 157.3907,-160 160.0227,-160 162.6546,-162.6319 162.6546,-165.2639 162.6546,-165.2639 162.6546,-176.7361 162.6546,-176.7361 162.6546,-179.3681 160.0227,-182 157.3907,-182"/>
    <text text-anchor="middle" x="154.7588" y="-166.8" font-family="Times,serif" font-size="14.00" fill="#000000">+</text>
    </g>
    <!-- &#45;5099837602000299771&#45;&gt;&#45;4827002429087198528 -->
    <g id="edge3" class="edge">
    <title>&#45;5099837602000299771&#45;&gt;&#45;4827002429087198528</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M154.7588,-199.6446C154.7588,-194.1937 154.7588,-187.6819 154.7588,-182.2453"/>
    </g>
    <!-- &#45;5099836340800970850 -->
    <g id="node6" class="node">
    <title>&#45;5099836340800970850</title>
    <path fill="none" stroke="#000000" stroke-width=".25" d="M191.4447,-182C191.4447,-182 186.0729,-182 186.0729,-182 183.387,-182 180.7011,-179.3141 180.7011,-176.6282 180.7011,-176.6282 180.7011,-165.3718 180.7011,-165.3718 180.7011,-162.6859 183.387,-160 186.0729,-160 186.0729,-160 191.4447,-160 191.4447,-160 194.1306,-160 196.8165,-162.6859 196.8165,-165.3718 196.8165,-165.3718 196.8165,-176.6282 196.8165,-176.6282 196.8165,-179.3141 194.1306,-182 191.4447,-182"/>
    <text text-anchor="middle" x="188.7588" y="-166.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
    </g>
    <!-- &#45;5099837602000299771&#45;&gt;&#45;5099836340800970850 -->
    <g id="edge4" class="edge">
    <title>&#45;5099837602000299771&#45;&gt;&#45;5099836340800970850</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M162.9892,-201.3171C168.3349,-195.0281 175.2736,-186.8649 180.6041,-180.5937"/>
    </g>
    <!-- 2460681415253629042 -->
    <!-- &#45;8751665616789673910&#45;&gt;2460681415253629042 -->
    <!-- &#45;5099836340803300948&#45;&gt;&#45;4827002429087198528 -->
    <!-- 7232982101612064143 -->
    <g id="node8" class="node">
    <title>7232982101612064143</title>
    <path fill="none" stroke="#000000" stroke-width="1.25" d="M120.7222,-142C120.7222,-142 116.7954,-142 116.7954,-142 114.832,-142 112.8686,-140.0366 112.8686,-138.0732 112.8686,-138.0732 112.8686,-123.9268 112.8686,-123.9268 112.8686,-121.9634 114.832,-120 116.7954,-120 116.7954,-120 120.7222,-120 120.7222,-120 122.6856,-120 124.6489,-121.9634 124.6489,-123.9268 124.6489,-123.9268 124.6489,-138.0732 124.6489,-138.0732 124.6489,-140.0366 122.6856,-142 120.7222,-142"/>
    <text text-anchor="middle" x="118.7588" y="-126.8" font-family="Times,serif" font-size="14.00" fill="#000000">i</text>
    </g>
    <!-- &#45;5099836340803300948&#45;&gt;7232982101612064143 -->
    <g id="edge8" class="edge">
    <title>&#45;5099836340803300948&#45;&gt;7232982101612064143</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M120.191,-159.6446C119.9185,-154.1937 119.5929,-147.6819 119.3211,-142.2453"/>
    </g>
    <!-- &#45;4827002429087198528&#45;&gt;&#45;5099836340800970850 -->
    <!-- &#45;5099838863197298594 -->
    <g id="node10" class="node">
    <title>&#45;5099838863197298594</title>
    <path fill="none" stroke="#000000" stroke-width=".25" d="M157.4447,-102C157.4447,-102 152.0729,-102 152.0729,-102 149.387,-102 146.7011,-99.3141 146.7011,-96.6282 146.7011,-96.6282 146.7011,-85.3718 146.7011,-85.3718 146.7011,-82.6859 149.387,-80 152.0729,-80 152.0729,-80 157.4447,-80 157.4447,-80 160.1306,-80 162.8165,-82.6859 162.8165,-85.3718 162.8165,-85.3718 162.8165,-96.6282 162.8165,-96.6282 162.8165,-99.3141 160.1306,-102 157.4447,-102"/>
    <text text-anchor="middle" x="154.7588" y="-86.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
    </g>
    <!-- &#45;5099836340800970850&#45;&gt;&#45;5099838863197298594 -->
    <g id="edge10" class="edge">
    <title>&#45;5099836340800970850&#45;&gt;&#45;5099838863197298594</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M184.0312,-159.8763C177.5354,-144.5921 165.9208,-117.2636 159.4462,-102.0292"/>
    </g>
    <!-- &#45;401902271210389261 -->
    <g id="node11" class="node">
    <title>&#45;401902271210389261</title>
    <path fill="none" stroke="#000000" stroke-width="1.25" d="M193.2588,-102C193.2588,-102 188.2588,-102 188.2588,-102 185.7588,-102 183.2588,-99.5 183.2588,-97 183.2588,-97 183.2588,-85 183.2588,-85 183.2588,-82.5 185.7588,-80 188.2588,-80 188.2588,-80 193.2588,-80 193.2588,-80 195.7588,-80 198.2588,-82.5 198.2588,-85 198.2588,-85 198.2588,-97 198.2588,-97 198.2588,-99.5 195.7588,-102 193.2588,-102"/>
    <text text-anchor="middle" x="190.7588" y="-86.8" font-family="Times,serif" font-size="14.00" fill="#000000">*</text>
    </g>
    <!-- &#45;5099836340800970850&#45;&gt;&#45;401902271210389261 -->
    <g id="edge11" class="edge">
    <title>&#45;5099836340800970850&#45;&gt;&#45;401902271210389261</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M189.0369,-159.8763C189.419,-144.5921 190.1022,-117.2636 190.4831,-102.0292"/>
    </g>
    <!-- &#45;5099838863199628692 -->
    <g id="node12" class="node">
    <title>&#45;5099838863199628692</title>
    <path fill="none" stroke="#000000" stroke-width=".25" d="M229.4447,-102C229.4447,-102 224.0729,-102 224.0729,-102 221.387,-102 218.7011,-99.3141 218.7011,-96.6282 218.7011,-96.6282 218.7011,-85.3718 218.7011,-85.3718 218.7011,-82.6859 221.387,-80 224.0729,-80 224.0729,-80 229.4447,-80 229.4447,-80 232.1306,-80 234.8165,-82.6859 234.8165,-85.3718 234.8165,-85.3718 234.8165,-96.6282 234.8165,-96.6282 234.8165,-99.3141 232.1306,-102 229.4447,-102"/>
    <text text-anchor="middle" x="226.7588" y="-86.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
    </g>
    <!-- &#45;5099836340800970850&#45;&gt;&#45;5099838863199628692 -->
    <g id="edge12" class="edge">
    <title>&#45;5099836340800970850&#45;&gt;&#45;5099838863199628692</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M194.7416,-159.7778C197.5475,-154.4285 200.8958,-147.9192 203.7588,-142 210.3564,-128.3596 217.4539,-112.4052 221.9922,-102.0222"/>
    </g>
    <!-- &#45;5330952088841566707 -->
    <!-- 2460681415253629042&#45;&gt;&#45;5330952088841566707 -->
    <!-- 8081897518576490873 -->
    <!-- &#45;5330952088841566707&#45;&gt;8081897518576490873 -->
    <!-- &#45;5099838863197298594&#45;&gt;&#45;401902271210389261 -->
    <!-- 7232979579218066497 -->
    <g id="node14" class="node">
    <title>7232979579218066497</title>
    <path fill="none" stroke="#000000" stroke-width="1.25" d="M156.7222,-62C156.7222,-62 152.7954,-62 152.7954,-62 150.832,-62 148.8686,-60.0366 148.8686,-58.0732 148.8686,-58.0732 148.8686,-43.9268 148.8686,-43.9268 148.8686,-41.9634 150.832,-40 152.7954,-40 152.7954,-40 156.7222,-40 156.7222,-40 158.6856,-40 160.6489,-41.9634 160.6489,-43.9268 160.6489,-43.9268 160.6489,-58.0732 160.6489,-58.0732 160.6489,-60.0366 158.6856,-62 156.7222,-62"/>
    <text text-anchor="middle" x="154.7588" y="-46.8" font-family="Times,serif" font-size="14.00" fill="#000000">i</text>
    </g>
    <!-- &#45;5099838863197298594&#45;&gt;7232979579218066497 -->
    <g id="edge16" class="edge">
    <title>&#45;5099838863197298594&#45;&gt;7232979579218066497</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M154.7588,-79.6446C154.7588,-74.1937 154.7588,-67.6819 154.7588,-62.2453"/>
    </g>
    <!-- &#45;401902271210389261&#45;&gt;&#45;5099838863199628692 -->
    <!-- 7232980840415065320 -->
    <g id="node16" class="node">
    <title>7232980840415065320</title>
    <path fill="none" stroke="#000000" stroke-width="1.25" d="M228.7222,-22C228.7222,-22 224.7954,-22 224.7954,-22 222.832,-22 220.8686,-20.0366 220.8686,-18.0732 220.8686,-18.0732 220.8686,-3.9268 220.8686,-3.9268 220.8686,-1.9634 222.832,0 224.7954,0 224.7954,0 228.7222,0 228.7222,0 230.6856,0 232.6489,-1.9634 232.6489,-3.9268 232.6489,-3.9268 232.6489,-18.0732 232.6489,-18.0732 232.6489,-20.0366 230.6856,-22 228.7222,-22"/>
    <text text-anchor="middle" x="226.7588" y="-6.8" font-family="Times,serif" font-size="14.00" fill="#000000">i</text>
    </g>
    <!-- &#45;5099838863199628692&#45;&gt;7232980840415065320 -->
    <g id="edge18" class="edge">
    <title>&#45;5099838863199628692&#45;&gt;7232980840415065320</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M226.7588,-79.8763C226.7588,-64.5921 226.7588,-37.2636 226.7588,-22.0292"/>
    </g>
    <!-- &#45;6081948820568943662 -->
    <!-- 8081897518576490873&#45;&gt;&#45;6081948820568943662 -->
    </g>
    </svg>
     <?xml version="1.0" encoding="UTF-8" standalone="no"?>
    <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
     "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
    <!-- Generated by graphviz version 2.40.1 (20161225.0304)
     -->
    <!-- Title: %3 Pages: 1 -->
    <svg width="239pt" height="230pt"
     viewBox="0.00 0.00 239.03 230.00" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
    <g id="graph0" class="graph" transform="scale(1 1) rotate(0) translate(4 226)">
    <title>%3</title>
    <polygon fill="#ffffff" stroke="transparent" points="-4,4 -4,-226 235.0347,-226 235.0347,4 -4,4"/>
    <!-- &#45;5493785780082770843 -->
    <!-- &#45;8751665616789673910 -->
    <!-- &#45;5493785780082770843&#45;&gt;&#45;8751665616789673910 -->
    <!-- &#45;5099837602000299771 -->
    <g id="node2" class="node">
    <title>&#45;5099837602000299771</title>
    <path fill="none" stroke="#000000" stroke-width=".25" d="M191.4447,-222C191.4447,-222 186.0729,-222 186.0729,-222 183.387,-222 180.7011,-219.3141 180.7011,-216.6282 180.7011,-216.6282 180.7011,-205.3718 180.7011,-205.3718 180.7011,-202.6859 183.387,-200 186.0729,-200 186.0729,-200 191.4447,-200 191.4447,-200 194.1306,-200 196.8165,-202.6859 196.8165,-205.3718 196.8165,-205.3718 196.8165,-216.6282 196.8165,-216.6282 196.8165,-219.3141 194.1306,-222 191.4447,-222"/>
    <text text-anchor="middle" x="188.7588" y="-206.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
    </g>
    <!-- &#45;5099836340803300948 -->
    <g id="node4" class="node">
    <title>&#45;5099836340803300948</title>
    <path fill="none" stroke="#000000" stroke-width=".25" d="M157.4447,-182C157.4447,-182 152.0729,-182 152.0729,-182 149.387,-182 146.7011,-179.3141 146.7011,-176.6282 146.7011,-176.6282 146.7011,-165.3718 146.7011,-165.3718 146.7011,-162.6859 149.387,-160 152.0729,-160 152.0729,-160 157.4447,-160 157.4447,-160 160.1306,-160 162.8165,-162.6859 162.8165,-165.3718 162.8165,-165.3718 162.8165,-176.6282 162.8165,-176.6282 162.8165,-179.3141 160.1306,-182 157.4447,-182"/>
    <text text-anchor="middle" x="154.7588" y="-166.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
    </g>
    <!-- &#45;5099837602000299771&#45;&gt;&#45;5099836340803300948 -->
    <g id="edge2" class="edge">
    <title>&#45;5099837602000299771&#45;&gt;&#45;5099836340803300948</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M180.5283,-201.3171C175.1827,-195.0281 168.2439,-186.8649 162.9134,-180.5937"/>
    </g>
    <!-- &#45;401904793604386907 -->
    <g id="node5" class="node">
    <title>&#45;401904793604386907</title>
    <path fill="none" stroke="#000000" stroke-width="1.25" d="M191.2588,-182C191.2588,-182 186.2588,-182 186.2588,-182 183.7588,-182 181.2588,-179.5 181.2588,-177 181.2588,-177 181.2588,-165 181.2588,-165 181.2588,-162.5 183.7588,-160 186.2588,-160 186.2588,-160 191.2588,-160 191.2588,-160 193.7588,-160 196.2588,-162.5 196.2588,-165 196.2588,-165 196.2588,-177 196.2588,-177 196.2588,-179.5 193.7588,-182 191.2588,-182"/>
    <text text-anchor="middle" x="188.7588" y="-166.8" font-family="Times,serif" font-size="14.00" fill="#000000">*</text>
    </g>
    <!-- &#45;5099837602000299771&#45;&gt;&#45;401904793604386907 -->
    <g id="edge3" class="edge">
    <title>&#45;5099837602000299771&#45;&gt;&#45;401904793604386907</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M188.7588,-199.6446C188.7588,-194.1937 188.7588,-187.6819 188.7588,-182.2453"/>
    </g>
    <!-- &#45;5099836340800970850 -->
    <g id="node6" class="node">
    <title>&#45;5099836340800970850</title>
    <path fill="none" stroke="#000000" stroke-width=".25" d="M225.4447,-182C225.4447,-182 220.0729,-182 220.0729,-182 217.387,-182 214.7011,-179.3141 214.7011,-176.6282 214.7011,-176.6282 214.7011,-165.3718 214.7011,-165.3718 214.7011,-162.6859 217.387,-160 220.0729,-160 220.0729,-160 225.4447,-160 225.4447,-160 228.1306,-160 230.8165,-162.6859 230.8165,-165.3718 230.8165,-165.3718 230.8165,-176.6282 230.8165,-176.6282 230.8165,-179.3141 228.1306,-182 225.4447,-182"/>
    <text text-anchor="middle" x="222.7588" y="-166.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
    </g>
    <!-- &#45;5099837602000299771&#45;&gt;&#45;5099836340800970850 -->
    <g id="edge4" class="edge">
    <title>&#45;5099837602000299771&#45;&gt;&#45;5099836340800970850</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M196.9892,-201.3171C202.3349,-195.0281 209.2736,-186.8649 214.6041,-180.5937"/>
    </g>
    <!-- 2460681415253629042 -->
    <!-- &#45;8751665616789673910&#45;&gt;2460681415253629042 -->
    <!-- &#45;5099836340803300948&#45;&gt;&#45;401904793604386907 -->
    <!-- &#45;5099840124394297417 -->
    <g id="node8" class="node">
    <title>&#45;5099840124394297417</title>
    <path fill="none" stroke="#000000" stroke-width=".25" d="M122.4447,-142C122.4447,-142 117.0729,-142 117.0729,-142 114.387,-142 111.7011,-139.3141 111.7011,-136.6282 111.7011,-136.6282 111.7011,-125.3718 111.7011,-125.3718 111.7011,-122.6859 114.387,-120 117.0729,-120 117.0729,-120 122.4447,-120 122.4447,-120 125.1306,-120 127.8165,-122.6859 127.8165,-125.3718 127.8165,-125.3718 127.8165,-136.6282 127.8165,-136.6282 127.8165,-139.3141 125.1306,-142 122.4447,-142"/>
    <text text-anchor="middle" x="119.7588" y="-126.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
    </g>
    <!-- &#45;5099836340803300948&#45;&gt;&#45;5099840124394297417 -->
    <g id="edge8" class="edge">
    <title>&#45;5099836340803300948&#45;&gt;&#45;5099840124394297417</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M146.6411,-161.7227C140.9868,-155.2606 133.4771,-146.6781 127.8324,-140.227"/>
    </g>
    <!-- &#45;4826998645498532157 -->
    <g id="node9" class="node">
    <title>&#45;4826998645498532157</title>
    <path fill="none" stroke="#000000" stroke-width="1.25" d="M156.3907,-142C156.3907,-142 151.1268,-142 151.1268,-142 148.4949,-142 145.8629,-139.3681 145.8629,-136.7361 145.8629,-136.7361 145.8629,-125.2639 145.8629,-125.2639 145.8629,-122.6319 148.4949,-120 151.1268,-120 151.1268,-120 156.3907,-120 156.3907,-120 159.0227,-120 161.6546,-122.6319 161.6546,-125.2639 161.6546,-125.2639 161.6546,-136.7361 161.6546,-136.7361 161.6546,-139.3681 159.0227,-142 156.3907,-142"/>
    <text text-anchor="middle" x="153.7588" y="-126.8" font-family="Times,serif" font-size="14.00" fill="#000000">+</text>
    </g>
    <!-- &#45;5099836340803300948&#45;&gt;&#45;4826998645498532157 -->
    <g id="edge9" class="edge">
    <title>&#45;5099836340803300948&#45;&gt;&#45;4826998645498532157</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M154.4749,-159.6446C154.3386,-154.1937 154.1758,-147.6819 154.0399,-142.2453"/>
    </g>
    <!-- &#45;5099840124391967319 -->
    <g id="node10" class="node">
    <title>&#45;5099840124391967319</title>
    <path fill="none" stroke="#000000" stroke-width=".25" d="M190.4447,-142C190.4447,-142 185.0729,-142 185.0729,-142 182.387,-142 179.7011,-139.3141 179.7011,-136.6282 179.7011,-136.6282 179.7011,-125.3718 179.7011,-125.3718 179.7011,-122.6859 182.387,-120 185.0729,-120 185.0729,-120 190.4447,-120 190.4447,-120 193.1306,-120 195.8165,-122.6859 195.8165,-125.3718 195.8165,-125.3718 195.8165,-136.6282 195.8165,-136.6282 195.8165,-139.3141 193.1306,-142 190.4447,-142"/>
    <text text-anchor="middle" x="187.7588" y="-126.8" font-family="Times,serif" font-size="14.00" fill="#000000">E</text>
    </g>
    <!-- &#45;5099836340803300948&#45;&gt;&#45;5099840124391967319 -->
    <g id="edge10" class="edge">
    <title>&#45;5099836340803300948&#45;&gt;&#45;5099840124391967319</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M163.0861,-160.9063C168.1268,-154.7963 174.529,-147.0361 179.5485,-140.9519"/>
    </g>
    <!-- &#45;401904793604386907&#45;&gt;&#45;5099836340800970850 -->
    <!-- 7232980840415065320 -->
    <g id="node16" class="node">
    <title>7232980840415065320</title>
    <path fill="none" stroke="#000000" stroke-width="1.25" d="M226.7222,-22C226.7222,-22 222.7954,-22 222.7954,-22 220.832,-22 218.8686,-20.0366 218.8686,-18.0732 218.8686,-18.0732 218.8686,-3.9268 218.8686,-3.9268 218.8686,-1.9634 220.832,0 222.7954,0 222.7954,0 226.7222,0 226.7222,0 228.6856,0 230.6489,-1.9634 230.6489,-3.9268 230.6489,-3.9268 230.6489,-18.0732 230.6489,-18.0732 230.6489,-20.0366 228.6856,-22 226.7222,-22"/>
    <text text-anchor="middle" x="224.7588" y="-6.8" font-family="Times,serif" font-size="14.00" fill="#000000">i</text>
    </g>
    <!-- &#45;5099836340800970850&#45;&gt;7232980840415065320 -->
    <g id="edge18" class="edge">
    <title>&#45;5099836340800970850&#45;&gt;7232980840415065320</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M222.8994,-159.7491C223.2675,-130.3006 224.256,-51.2201 224.621,-22.026"/>
    </g>
    <!-- &#45;5330952088841566707 -->
    <!-- 2460681415253629042&#45;&gt;&#45;5330952088841566707 -->
    <!-- &#45;5099840124394297417&#45;&gt;&#45;4826998645498532157 -->
    <!-- 7232983362809062966 -->
    <g id="node12" class="node">
    <title>7232983362809062966</title>
    <path fill="none" stroke="#000000" stroke-width="1.25" d="M121.7222,-102C121.7222,-102 117.7954,-102 117.7954,-102 115.832,-102 113.8686,-100.0366 113.8686,-98.0732 113.8686,-98.0732 113.8686,-83.9268 113.8686,-83.9268 113.8686,-81.9634 115.832,-80 117.7954,-80 117.7954,-80 121.7222,-80 121.7222,-80 123.6856,-80 125.6489,-81.9634 125.6489,-83.9268 125.6489,-83.9268 125.6489,-98.0732 125.6489,-98.0732 125.6489,-100.0366 123.6856,-102 121.7222,-102"/>
    <text text-anchor="middle" x="119.7588" y="-86.8" font-family="Times,serif" font-size="14.00" fill="#000000">i</text>
    </g>
    <!-- &#45;5099840124394297417&#45;&gt;7232983362809062966 -->
    <g id="edge14" class="edge">
    <title>&#45;5099840124394297417&#45;&gt;7232983362809062966</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M119.7588,-119.6446C119.7588,-114.1937 119.7588,-107.6819 119.7588,-102.2453"/>
    </g>
    <!-- &#45;4826998645498532157&#45;&gt;&#45;5099840124391967319 -->
    <!-- 7232979579218066497 -->
    <g id="node14" class="node">
    <title>7232979579218066497</title>
    <path fill="none" stroke="#000000" stroke-width="1.25" d="M188.7222,-62C188.7222,-62 184.7954,-62 184.7954,-62 182.832,-62 180.8686,-60.0366 180.8686,-58.0732 180.8686,-58.0732 180.8686,-43.9268 180.8686,-43.9268 180.8686,-41.9634 182.832,-40 184.7954,-40 184.7954,-40 188.7222,-40 188.7222,-40 190.6856,-40 192.6489,-41.9634 192.6489,-43.9268 192.6489,-43.9268 192.6489,-58.0732 192.6489,-58.0732 192.6489,-60.0366 190.6856,-62 188.7222,-62"/>
    <text text-anchor="middle" x="186.7588" y="-46.8" font-family="Times,serif" font-size="14.00" fill="#000000">i</text>
    </g>
    <!-- &#45;5099840124391967319&#45;&gt;7232979579218066497 -->
    <g id="edge16" class="edge">
    <title>&#45;5099840124391967319&#45;&gt;7232979579218066497</title>
    <path fill="none" stroke="#000000" stroke-width=".5" d="M187.6197,-119.8763C187.4287,-104.5921 187.0871,-77.2636 186.8967,-62.0292"/>
    </g>
    <!-- 8081897518576490873 -->
    <!-- &#45;5330952088841566707&#45;&gt;8081897518576490873 -->
    <!-- &#45;6081948820568943662 -->
    <!-- 8081897518576490873&#45;&gt;&#45;6081948820568943662 -->
    </g>
    </svg>
    </div>

