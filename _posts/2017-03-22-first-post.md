---
layout: post
title:  "Scipy distance and rdkit fingerprints"
date:   2017-03-22 18:10:21 +0000
categories: chemoinformatics
---
This is the test first post of the blog. As an example I am using a question that was sent to the rdkit users email list. The question was regarding comparing bit vectors as numpy arrays. The great thing about scipy and rdkit is that you don't need to have to convert between the fingerprints and numpy arrays.

For example:
{% highlight python %}
from __future__ import print_function
from rdkit import Chem
from rdkit.Chem import AllChem
from scipy.spatial import distance


mol1 = Chem.MolFromSmiles('CCO')
mol2 = Chem.MolFromSmiles('CCC')

fp1 = AllChem.GetMorganFingerprintAsBitVect(mol1, 8)
fp2 = AllChem.GetMorganFingerprintAsBitVect(mol2, 8)

# jaccard distance is the same as tanimoto distance
# 1 - distance = similarity
print(1 - distance.jaccard(fp1, fp2))

# prints 0.4285714285714286

{% endhighlight %}

This takes advantage of the fact that jaccard distance is the same as the tanimoto distance and that the similarity is 1 - distance.

As Greg responded in his email to the list, there is already a function on rdkit that does this. So why do we care? The reason is that it can be incorporated in a workflow that creates efficient distance matrices in scipy.

Let's have a look at a quick example:

{% highlight python %}
from rdkit import Chem
from rdkit.Chem import AllChem
from scipy.spatial import distance

sdf_file = 'path/to/sdf_file'

# rdkit mol and fingerprint generators
mols = (x for x in Chem.SDMolSupplier(sdf_file) if x is not None)
fps = [AllChem.GetMorganFingerprintAsBitVect(m, 8) for m in mols]

# using scipy distance matrix to compare the similarity of all mols
S = 1 - distance.pdist(fps, metric='jaccard')
{% endhighlight %}

The result is a compressed distance matrix of the similarities of all of our molecules, which can be used as the input for clustering. Otherwise, we can also use cdist to compare a list of reference molecules against a database. This can then be used un virtual screening.
