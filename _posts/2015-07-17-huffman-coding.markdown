---
layout: post
title:  "Codage de Huffman"
date:   2015-07-17
categories:  "informatique"
---

# Principe

# Utilisation

En bionformatique, un grand nombre d'applications utilisent les concepts liés à l'analyse de texte (algorithimique du texte, stringology). Cela s'accompagne de problématique de vitesse de traitement de l'information, mais aussi de consommation mémoire (physique et virtuelle). Le codage de Huffman est une méthode largement utlisée dans les outils de bioinformatique. La compréhension de son fonctionnement est donc intéressante pour appréhender les méthodes d'indexation de données par exemple. Pour illustrer son utilisation, nous pouvons citer quelques publications comme ...

# Algorithme





# Example d'implementation

{% highlight python %}

import struct
from operator import attrgetter
import cPickle as pickle  

class Huffman(object):
  
  def __init__(self, string=None):
    self.string = string
    self._bitmap = {}
    self._rootTree = None
    
  def encode(self):
    if self.string:
      self._getTree()
      self._assignRecursiveBitNode(self._bitmap, self._rootTree)
      self._convertToBinary()
      #self.decode(buff)
    else:
      raise Exception("no string to encode")

  def write(self,filename):
    f = open(filename, "wb")
    pickle.dump(self._rootTree, f)
    f.write("%s\n" % self._totalLength)
    f.write(self._barray)
    f.close()
    
  def read(self,filename):
    f = open(filename,'r')
    self._newRootTree = pickle.load(f)    # size in bytes
    self._newLength = int(f.readline().rstrip())
    self._newBarray = f.readline()
   
    
  def decode(self):
    self._convertFromBinary()
    
    
  def _getCharFreqFromString(self, s):
    dFrequencies = {}
    for i in s:
      if i in dFrequencies:
        dFrequencies[i] += 1
      else:
        dFrequencies[i] = 1
    return dFrequencies
  
  def _getTree(self):
    dFreqs = self._getCharFreqFromString(self.string)
    lnodes = [Node(ch=ch, fq=fq) for ch,fq in dFreqs.iteritems()]
    self._rootTree =  self._buildRecursiveTree(lnodes)
    return

  def _convertToBinary(self):
    BITS = 8
    barray = bytearray()
    buff = length = totalLength =  0
    for c in self.string:
      bitnode = self._bitmap[c]
      for bit in bitnode:
        if bit == '1':
          buff = (buff << 1) | 0x01
        else:
          buff = (buff << 1)
        length += 1 
        print length
        if length == BITS :
          barray.extend(struct.pack("B",buff))
          totalLength += length
          buff = length = 0      
        
    if length != 0 :
      barray.extend(struct.pack("B", buff << (BITS-length)))
      totalLength += length
  
    self._totalLength = totalLength
    self._barray = barray
    
  def _convertFromBinary(self):
    BITS = 8
    totalLength = 0
    currentNode = self._newRootTree    
    buff = []
    for ch in self._newBarray:
      buffLength = 0
      while buffLength < BITS and totalLength != self._newLength:
        buffLength += 1
        totalLength += 1
        bits = struct.unpack('B',ch)[0]
        
        if bits >> (BITS-buffLength) & 1 :
          currentNode = currentNode._rchild
          if currentNode._ch:
            buff.append(currentNode._ch)
            currentNode = self._newRootTree                   
        else:
          currentNode = currentNode._lchild
          if currentNode._ch:
            buff.append(currentNode._ch)
            currentNode = self._newRootTree
            
      print ''.join(buff)  

# on lit a lenvers les bit pour pouvoir descndre larbre
    
    
  def _buildRecursiveTree(self, lnodes):
    sortedLnodes = sorted(lnodes, key=attrgetter('_fq') , reverse = False)
    print sortedLnodes
    child1 = sortedLnodes.pop(0)
    child2 = sortedLnodes.pop(0)
    parent = Node(lchild=child1, rchild=child2, fq=child1._fq+child2._fq) 
    child1._parent = child2._parent = parent
    sortedLnodes.append(parent)
    if len(sortedLnodes) > 1:
      return self._buildRecursiveTree(sortedLnodes)
    else:
      return parent
    
  def _assignRecursiveBitNode(self, bitmap, node, bitcode=''):
    #print node
    #print bitmap
    if not node._lchild and not node._rchild:
      bitmap[node._ch] = bitcode
      print "%s : %s" % (node._ch, bitcode)
      return
    self._assignRecursiveBitNode(bitmap, node._lchild, ("%s0" % bitcode))
    self._assignRecursiveBitNode(bitmap, node._rchild, ("%s1" % bitcode))

 
class Node(object):

  def __init__(self, parent=None, lchild=None, rchild=None, fq=None, ch=None):
    self._parent=parent
    self._lchild=lchild
    self._rchild=rchild
    self._fq=int(fq)
    self._ch=ch
    
  def __repr__(self):
    return '(%s:%s:%d)'%(self._ch, self._lchild,self._fq)

    
if __name__=='__main__':
  #iHuffman = Huffman('wikipedia')
  iHuffman = Huffman('atgcnatgcnatgcna')
  iHuffman.encode()
  iHuffman.write("test.out.wr")
  iHuffman.read("test.out.wr")
  iHuffman.decode()
  
  
# algoritme pour construire l'arbre
# pour l'ensemble des lettres je cree un noeud avec la frequence
# je trie les noeuds par frequence 
# je regroupe les 2 noeuds les plus faible et cree 1 noeud parent
# je supprime les 2 noeuds fils

{% endhighlight %}


http://genome.cshlp.org/content/21/5/734.full

Huffman D. 1952. A method for the construction of minimum redundancy codes. In Proceedings of the I.R.E., Vol. 40, No. 9, pp. 1098–1101. Massachusetts Institute of Technology, Cambridge, MA.
	

http://bioinformatics.oxfordjournals.org/content/25/2/274.fullI
