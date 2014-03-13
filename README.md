# blproof - Blind liability proof

[![Build Status](https://travis-ci.org/olalonde/blind-liability-proof.png)](https://travis-ci.org/olalonde/blind-liability-proof)

Intended for use as part of the
[olalonde/blind-solvency-proof](https://github.com/olalonde/blind-solvency-proof)
scheme.

Beer fund: **1ECyyu39RtDNAuk3HRCRWwD4syBF2ZGzdx**

## Install

[Install Node.js](http://nodejs.org/)

```
npm install -g blproof
```

## CLI Usage

Simple usage:

```
# Generate a partial tree for all users in accounts.json.
# Partial trees will be saved to partial_trees/ directory.
# complete_tree.json and root.json will be saved to current directory.
# For a sample accounts file, refer to test/accounts.json.

$ blproof generate -f accounts.json

# Verify a partial tree 

$ blproof verify --root root.json -f partial_trees/satoshi.json

# or (where hash is the root hash and value is the root value)

$ blproof verify --hash "SLpDal8kYJNdLwczp6wrU68FOFrpoHT3w5nd15HOpwU=" --value 37618 -f mark.json
```

Advanced usage: 

```
# Create complete proof tree from an accounts file (see
# test/account.json for format)

$ blproof completetree -f test/accounts.json --human
$ blproof completetree -f test/accounts.json > complete.out.json

# Extract partial tree for user mark.

$ blproof partialtree mark -f complete.out.json --human
$ blproof partialtree mark -f complete.out.json > mark.out.json

# Display root node hash and value

$ blproof root -f complete.out.json --human

# Verify partial tree

$ blproof verify --hash "SLpDal8kYJNdLwczp6wrU68FOFrpoHT3w5nd15HOpwU=" --value 37618 -f mark.out.json
```

## Library usage

See `cli.js`. 

Browser build: `browserify index.js --standalone blproof > build/blproof.js`.

## Definitions

### Complete proof tree
  
The complete proof tree is a binary tree where the leaf nodes
represent all the user accounts and the interior nodes generated using the
NodeCombiner function described below.

The complete tree should be kept private by the operator in order to
protect the privacy of its users. Only the root node should be puslished
publicly and each individual user should have private access to their
own partial proof tree.

### Interior node (NodeCombiner function)

Interior nodes are generated using the NodeCombiner function described
below.

The node's value is equal to the sum of its two child node's values.

The node's hash is equal to the sha256 of its value concatenated with
its child's hashes.

```
function NodeCombiner (left_node, right_node) {
  var n = {};
  n.value = left_node.value + right_node.value;
  n.hash = sha256((left_node.value + right_node.value) + '' + left_node.hash + '' + right_node.hash);
  return n;
}
```

### Leaf node

Leaf nodes represent user accounts. They possess the following values:

  - `user`: A unique identifier for the user. It is necessary for a user
    to assess the uniqueness of this value so it is recommended to use their username or email.
  - `nonce`: A random number to prevent its neighboor node from discovering its `user` value
  - `value`: The user's balance.
  - `hash`: sha256(user + '|' + value + '|' + nonce)


### Root node

The root node of the tree like all interior nodes possesses a hash and a
value. This data must be published publicly as a way to prove that all users
are part of the same proof tree.


### Partial proof tree

A partial proof contains only the nodes from the complete root a given
user needs to verify he was included in the tree. 

It can be generated by starting from the user's leaf node and moving up
the tree until reaching the root node. Then the sibblings of each
selected node on the path must be added to the tree. 

- All internal nodes should be completely stripped of their `data` since they need to
be computed during verification.

- All leaf nodes should be stripped of their `user` and `nonce` properties
except for the leaf representing the current user.

- The leaf representing the current `user` should be stripped of its hash
property since it needs to be computed during verification.

Partial trees should be disclosed privately to each individual users so
they can verify the proof.

## Serialized data formats (work in progress / draft)

This section is intended to standardize the way root nodes and trees are
distributed in order to make implementations compatible. 

All formats are based on JSON. 

### Root node:

```javascript
{ 
  "root": {
    "value": 37618,
    "hash": "2evVTMS8wbF2p5aq1qFETanO24BsnP/eshJxxPHJcug="
  }
}
```

Hash is the sha256 hash in base64 encoding. Value is a number (float or integer).

### Partial trees:

Partial trees are represented as a `node` object graph. Nodes have the following format:

```javascript
{
  "left": <node>,
  "right": <node>,
  "data": <node_data>
}
```

`<node_data>` is an object which must contain the following keys:

- `value` number
- `hash` string SHA256 hash in base64 encoding
- `user` string (optional) Only the node belonging to the user this partial tree
  was generated for should have this key set.
- `nonce` number (optional) Only the node belonging to the user this partial tree
  was generated for should have this key set.

## Publishing format 

See [olalonde/blind-solvency-proof](https://github.com/olalonde/blind-solvency-proof#assets-proof).

## Some sample outputs

In `test/`. Note that the hash values change when executing the same
command twice as the nonce are generated randomly.

```bash

$ ../cli.js completetree -f accounts.json --human

37618, dLHSG4ZyIxZ/f4R7XtJ+pVIqtsX9AMY5ivPmaUGu+Sg=
 |_ 24614, hkqF8w5tv9lifzRrJW/hKn7Q3ANrr3t751gFARu6IeY=
 | |_ 21072, ScXstoH3/way/cWYISRyLsXd2aJTqzmfuj101VqsivY=
 | | |_ 4167, dWhYurpOonzMw+7uAJDAhEkKHXeKQdS3dCd5mH8U9PU=
 | | | |_ 122, 9KYbpT+YWEeizUQmNvY5LX2x6o+svzi4xZ3uI43xgVE=
 | | | | |_ 39, einstein, 0.09985585859976709
 | | | | |_ 83, picasso, 0.7243645829148591
 | | | |_ 4045, olalonde, 0.9388562678359449
 | | |_ 16905, 6qcgujpnznmgbpwroma3vQRVc34LyLKJDu9JHH4clSQ=
 | |   |_ 6905, gmaxwell, 0.7123587417881936
 | |   |_ 10000, satoshi, 0.46833383152261376
 | |_ 3542, xeVsix+vju6GsUf2iOVzp4MgH6wUIvqkkO63UcW7RDM=
 |   |_ 3327, cVcRdhHtDAjYPGKJgFeeQWiTTpURZ2hJ30iEM6xtvgA=
 |   | |_ 300, luke-jr, 0.47092385264113545
 |   | |_ 3027, sipa, 0.10589101002551615
 |   |_ 215, 0TCVjiuCgxilZAgXbeAoJT3/zoaYcvbZiy7MAL8ukrw=
 |     |_ 200, codeshark, 0.12324331048876047
 |     |_ 15, gribble, 0.9504100847989321
 |_ 13004, TFyQdFpnE2G9PEAf03HQ73FyDyRTy9kRL70kePdFo7k=
   |_ 9901, KxQ4gOBo6bHZA74FoIj7t5078+UQq+V71vC+MI+T4ck=
   | |_ 12, mno4J/cZ5bwol+VuRNWFUTR6uYWu/bND18/TFSISuMU=
   | | |_ 0, alice, 0.13097181799821556
   | | |_ 12, bob, 0.99190160818398
   | |_ 9889, VBFmIlvgxQ/jnk54CSlfHxrLcynzLbGKxjndiOsULmE=
   |   |_ 9427, charlie, 0.7431337819434702
   |   |_ 462, mark, 0.28908941335976124
   |_ 3103, c1YTJY7XXWVcFnHQz8k13jNTF3TpTswKNgJUTfR3OIg=
     |_ 3032, sTZW88KQc99WmN9DkgDAvmeqSrShe+tfmb4m8UQDRoI=
     | |_ 12, anax, 0.7174370898865163
     | |_ 3020, gavin, 0.18109926907345653
     |_ 71, LnHLEktE7+FR7Oca2jkUKbSOHEvMJVWlC0oKup12DsQ=
       |_ 68, stacy, 0.9329951496329159
       |_ 3, justin, 0.77584387245588

$ ../cli.js completetree -f accounts.json > complete.out.json

$ ../cli.js partialtree mark -f complete.out.json --human

 |_ 24614, XjUIfej5Vxd3iu9BXCoJJI7hVAQRrg0gTukaypRzxDU=
 |_
   |_
   | |_ 12, NncluiYssLFglDr21RrRlmOHkn7XpVflFDycoQJdWOM=
   | |_
   |   |_ 9427, Tijifd355WjyUdYDg/WUixo07wzNEGmXtx63VJNxff0=
   |   |_ 462, mark, 0.4003799057099968
   |_ 3103, Kz0j/ebNpCvHcwRk31STdWjqngIeMKNdBxG39GY2gtU=

$ ../cli.js partialtree mark -f complete.out.json > partial.out.json

$ ../cli.js root -f complete.out.json
{"root":{"value":37618,"hash":"RXICgKsrMJV4OBP709Adnk/LaLQ7nqpPUljCQdz3pBU="}}

$ ../cli.js verify --value 37618 --hash "RXICgKsrMJV4OBP709Adnk/LaLQ7nqpPUljCQdz3pBU=" -f partial.out.json
Partial tree verified successfuly!

User: mark
Balance: 462

```

## References

- https://iwilcox.me.uk/2014/proving-bitcoin-reserves
- [Reddit announcement](http://www.reddit.com/r/Bitcoin/comments/1yzil4/i_implemented_gmaxwells/)
- [HN post](https://news.ycombinator.com/item?id=7277865)
- [Example of how a shared wallet website could use the CLI](http://www.reddit.com/r/Bitcoin/comments/1yzil4/i_implemented_gmaxwells/cfp50ib)
