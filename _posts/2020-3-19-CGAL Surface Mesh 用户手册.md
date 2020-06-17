---
layout:     post
title:      CGAL：Surface mesh
subtitle:   Surface_mesh中文用户手册
date:       2020-03-19
author:     Cory
header-img: img/post-bg-debug.png
catalog: true
tags:
    - CGAL
    - 图形学
    - 用户手册
---


#User Manual 用户手册
该类[Surface_mesh](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html )是半边数据结构的实现，可用于表示多面体表面。它是CGAL包[Halfedge Data Structure](https://doc.cgal.org/4.13.2/Manual/packages.html#PkgHDSSummary)和[3D Polyhedral Surface](https://doc.cgal.org/4.13.2/Manual/packages.html#PkgPolyhedronSummary)的替代方案。主要区别在于它是基于索引的，而不是基于指针的。此外，将信息添加到顶点，半边，边和面的机制要简单得多，并且可以在运行时(runtime)完成，而不必在编译时完成。

因为数据结构使用整数索引作为顶点，半边，边和面的描述符，所以它的内存占用空间比基于64位指针的版本要低。由于索引是连续的，因此它们可以用作存储属性的向量的索引。

当元素被删除时，它们仅被标记为已删除，并且必须调用垃圾回收函数才能真正删除它们。

##1 用法（usage）
主类[Surface_mesh](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html "This class is a data structure that can be used as halfedge data structure or polyhedral surface...")提供了四个嵌套类，它们表示半边数据结构的基本元素：

- [Surface_mesh::Vertex_index](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh_1_1Vertex__index.html "This class represents a vertex. ")
- [Surface_mesh::Halfedge_index](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh_1_1Halfedge__index.html "This class represents a halfedge. ")
- [Surface_mesh::Face_index](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh_1_1Face__index.html "This class represents a face. ")
- [Surface_mesh::Edge_index](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh_1_1Edge__index.html "This class represents an edge. ")

这些class是基本图元个体的封装，这样做的目的是为了保证类型安全。它们默认是可构造的，因此可能会产生一些无效的元素。新的元素可以通过一系列不包括连通性的低级函数向surface mesh这个类添加或者删除。一个例外是[Surface_mesh::add_face()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#a3bbb4fcc69d61f1bd816a255b2251f74 "adds a new face, and resizes face properties if necessary. ")，它尝试向网格（由一系列顶点定义）中添加一个新面，如果操作在拓扑上无效，则失败。
```
typedef Surface_mesh<Point> Mesh;
Mesh m;
Mesh::Vertex_index u = m.add_vertex(Point(0,1,0));
Mesh::Vertex_index v = m.add_vertex(Point(0,0,0));
Mesh::Vertex_index w = m.add_vertex(Point(1,0,0));
m.add_face(u, v, w);
```
正如[Surface_mesh](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html "This class is a data structure that can be used as halfedge data structure or polyhedral surface...")是基于索引的(index based)，[Vertex_index](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh_1_1Vertex__index.html)，[Halfedge_index](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh_1_1Halfedge__index.html)，[Edge_index](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh_1_1Edge__index.html)和[Face_index](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh_1_1Face__index.html)没有成员函数来访问连接性或属性。从[Surface_mesh](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html "This class is a data structure that can be used as halfedge data structure or polyhedral surface...")中创建的函数必须有获取这些信息的功能。

##2 连通性（connectivity）
**surface mesh是一种能够保持顶点，边和面的入射信息的，以边为核心的数据结构。**
每个边由两个方向相反的半边表示，每个半边均存储对入射面和入射顶点的引用，此外，它还存储对入射面的下一个和上一个入射半边的引用。
其实说白了，surface mesh有一个半边结构的表达方式：对于一个半边h它有以下这样几个导航的表达方式：[Surface_mesh::opposite()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#aa7db4bc6d4c063059072b2f1a4609c0e "returns the opposite halfedge of h. Note that there is no function set_opposite(). ")，[Surface_mesh::next()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#aa1cc5db58c2a463d6e7dff79c8f01eda "returns the next halfedge within the incident face. ")，[Surface_mesh::prev()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#a59417a612605ec242f066bd9a1e28185 "returns the previous halfedge within the incident face. ")，[Surface_mesh::target()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#a1039dd1e0b038b526ddebe477e67f531 "returns the vertex the halfedge h points to. ")，和[Surface_mesh::face()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#a33419ae03fc4d9a8c28367dbe1241a21 "returns the face incident to halfedge h. ")和半边数据结构是差不多的。

##3 范围和迭代器（range and iterators）
[Surface_mesh](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html "This class is a data structure that can be used as halfedge data structure or polyhedral surface...")提供迭代器范围以枚举所有顶点，半边，边和面。它提供的成员函数返回与[Boost.Range](https://www.boost.org/libs/range/doc/html/index.html)库兼容的元素范围。

来自官方文档的示例,演示如何从范围中获取迭代器类型，获得begin和end迭代器的替代方法以及基于范围(range for c++11)的循环的替代方法。
```
#include <vector>

#include <boost/foreach.hpp>

#include <CGAL/Simple_cartesian.h>

#include <CGAL/Surface_mesh.h>

typedef CGAL::Simple_cartesian<double> K;

typedef CGAL::Surface_mesh<K::Point_3> Mesh;

typedef Mesh::Vertex_index vertex_descriptor;

typedef Mesh::Face_index face_descriptor;

int main()

{

Mesh m;

// u                x
// +------------+
// |            |
// |      f     |
// |            |
// +------------+
// v                w

// Add the points as vertices

vertex_descriptor u = m.add_vertex(K::Point_3(0,1,0));

vertex_descriptor v = m.add_vertex(K::Point_3(0,0,0));

vertex_descriptor w = m.add_vertex(K::Point_3(1,0,0));

vertex_descriptor x = m.add_vertex(K::Point_3(1,1,0));

/* face_descriptor f = */ m.add_face(u,v,w,x);

{

std::cout << "all vertices " << std::endl;

// The vertex iterator type is a nested type of the Vertex_range

Mesh::Vertex_range::iterator vb, ve;

Mesh::Vertex_range r = m.vertices();

// The iterators can be accessed through the C++ range API

vb = r.begin();

ve = r.end();

// or the boost Range API

vb = boost::begin(r);

ve = boost::end(r);

// or with boost::tie, as the CGAL range derives from std::pair

for(boost::tie(vb, ve) = m.vertices(); vb != ve; ++vb){

std::cout << *vb << std::endl;

}

// Instead of the classical for loop one can use

// the boost macro for a range

BOOST_FOREACH(vertex_descriptor vd, m.vertices()){

std::cout << vd << std::endl;

}

// or the C++11 for loop. Note that there is a ':' and not a ',' as in BOOST_FOREACH

#ifndef CGAL_CFG_NO_CPP0X_RANGE_BASED_FOR

for(vertex_descriptor vd : m.vertices()){

std::cout << vd << std::endl;

}

#endif

}

return 0;

}
```

##4 循环器（circulators）
[CGAL和Boost Graph Library](https://doc.cgal.org/4.13.2/Manual/packages.html#PkgBGLSummary)中提供了围绕面和顶点的循环器作为模板类。

围绕面的循环器基本上会进行调用[Surface_mesh::next()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#aa1cc5db58c2a463d6e7dff79c8f01eda "returns the next halfedge within the incident face. ")，以从该面的逆时针方向从半边变为半边，并且取消引用时返回半边或入射顶点或相对面。
- [CGAL::Halfedge_around_face_circulator](https://doc.cgal.org/4.13.2/BGL/classCGAL_1_1Halfedge__around__face__circulator.html)<Mesh>
- CGAL::Vertex_around_face_circulator<Mesh>
- [CGAL::Face_around_face_circulator](https://doc.cgal.org/4.13.2/BGL/classCGAL_1_1Face__around__face__circulator.html)<Mesh>

围绕目标边的顶点的循环器基本上会进行调用Surface_mesh::opposite(Surface_mesh::next())，以使围绕同一目标点的顺时针方向从半边变为半边。
- [CGAL::Halfedge_around_target_circulator](https://doc.cgal.org/4.13.2/BGL/classCGAL_1_1Halfedge__around__target__circulator.html)<Mesh>
- [CGAL::Vertex_around_target_circulator](https://doc.cgal.org/4.13.2/BGL/classCGAL_1_1Vertex__around__target__circulator.html)<Mesh>
- [CGAL::Face_around_target_circulator](https://doc.cgal.org/4.13.2/BGL/classCGAL_1_1Face__around__target__circulator.html)<Mesh>

###example:
下面的示例演示如何枚举给定半边目标周围的顶点。第二个循环表明，每种循环器类型都带有一个等效的迭代器和一个用于创建迭代器范围的自由函数
```
#include <vector>
#include <boost/foreach.hpp>
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Surface_mesh.h>
typedef CGAL::Simple_cartesian<double> K;
typedef CGAL::Surface_mesh<K::Point_3> Mesh;
typedef Mesh::Vertex_index vertex_descriptor;
typedef Mesh::Face_index face_descriptor;
int main()
{
  Mesh m;
  // u            x
  // +------------+
  // |            |
  // |            |
  // |      f     |
  // |            |
  // |            |
  // +------------+
  // v            w
  // Add the points as vertices
  vertex_descriptor u = m.add_vertex(K::Point_3(0,1,0));
  vertex_descriptor v = m.add_vertex(K::Point_3(0,0,0));
  vertex_descriptor w = m.add_vertex(K::Point_3(1,0,0));
  vertex_descriptor x = m.add_vertex(K::Point_3(1,1,0));
  face_descriptor f = m.add_face(u,v,w,x);
 
  {
    std::cout << "vertices around vertex " << v << std::endl;
    CGAL::Vertex_around_target_circulator<Mesh> vbegin(m.halfedge(v),m), done(vbegin);
    do {
      std::cout << *vbegin++ << std::endl;
    } while(vbegin != done);
  }
   
  { 
    std::cout << "vertices around face " << f << std::endl;
    CGAL::Vertex_around_face_iterator<Mesh> vbegin, vend;
    for(boost::tie(vbegin, vend) = vertices_around_face(m.halfedge(f), m);
        vbegin != vend; 
        ++vbegin){
      std::cout << *vbegin << std::endl;
    }
  }
  // or the same again, but directly with a range based loop
  BOOST_FOREACH(vertex_descriptor vd,vertices_around_face(m.halfedge(f), m)){
    std::cout << vd << std::endl;
  } 
    
  return 0;
}
```

##5 属性（properties）
[Surface_mesh](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html "This class is a data structure that can be used as halfedge data structure or polyhedral surface...")提供一种实时为顶点，半边，边和面指定新属性的机制。给定属性的所有值都存储为连续的内存块。每当将键类型的新元素添加到数据结构中或[Surface_mesh::collect_garbage()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#aea69dbef8122b1acf050c19063f935f2 "really removes vertices, halfedges, edges, and faces which are marked removed. ")执行功能时，对属性的引用都将无效。元素的属性在删除后将继续存在。尝试通过无效的元素访问属性将导致未定义的行为。
默认情况下，会维护一个属性"v:point"。通过将新点添加到数据结构时，必须提供此属性的值[Surface_mesh::add_vertex()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#a6f899386b0667edc64cfae79cc93386e "adds a new vertex, and resizes vertex properties if necessary. ")。可以使用[Surface_mesh::points()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#a0887ce92d0d65bcd1128c313bc9d2d07)或直接访问该属性`[Surface_mesh::point(Surface_mesh::Vertex_index v)](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#aaf6c8f0ab1ccc8a7b8b07de618aca2f8 "returns the point associated to vertex v. ")。

当元素被删除时，它仅被标记为已删除，并且在[Surface_mesh::collect_garbage()](https://doc.cgal.org/4.13.2/Surface_mesh/classCGAL_1_1Surface__mesh.html#aea69dbef8122b1acf050c19063f935f2 "really removes vertices, halfedges, edges, and faces which are marked removed. ")被调用时被真正删除。垃圾回收也确实会删除这些元素的属性。

连接性也存储在属性中，即名为“ v：connectivity”，“ h：connectivity”和“ f：connectivity”的属性。对于已删除元素的标记，它非常相似，其中有“ v：removed”，“ e：removed”和“ f：removed”。

###example：
```
#include <string>
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Surface_mesh.h>
#include <boost/foreach.hpp>
typedef CGAL::Simple_cartesian<double> K;
typedef CGAL::Surface_mesh<K::Point_3> Mesh;
typedef Mesh::Vertex_index vertex_descriptor;
typedef Mesh::Face_index face_descriptor;
int main()
{
  Mesh m;
  vertex_descriptor v0 = m.add_vertex(K::Point_3(0,2,0));
  vertex_descriptor v1 = m.add_vertex(K::Point_3(2,2,0));
  vertex_descriptor v2 = m.add_vertex(K::Point_3(0,0,0));
  vertex_descriptor v3 = m.add_vertex(K::Point_3(2,0,0));
  vertex_descriptor v4 = m.add_vertex(K::Point_3(1,1,0));
  m.add_face(v3, v1, v4);
  m.add_face(v0, v4, v1);
  m.add_face(v0, v2, v4);
  m.add_face(v2, v3, v4);
  // give each vertex a name, the default is empty
  Mesh::Property_map<vertex_descriptor,std::string> name;
  bool created;
  boost::tie(name, created) = m.add_property_map<vertex_descriptor,std::string>("v:name","");
  assert(created);
  // add some names to the vertices
  name[v0] = "hello";
  name[v2] = "world";
  {
    // You get an existing property, and created will be false
    Mesh::Property_map<vertex_descriptor,std::string> name;
    bool created;
    boost::tie(name, created) = m.add_property_map<vertex_descriptor,std::string>("v:name", "");
    assert(! created);
  }
  //  You can't get a property that does not exist
  Mesh::Property_map<face_descriptor,std::string> gnus;
  bool found;
  boost::tie(gnus, found) = m.property_map<face_descriptor,std::string>("v:gnus");
  assert(! found);
  // retrieve the point property for which exists a convenience function
  Mesh::Property_map<vertex_descriptor, K::Point_3> location = m.points();
  BOOST_FOREACH( vertex_descriptor vd, m.vertices()) { 
    std::cout << name[vd] << " @ " << location[vd] << std::endl;
  }
  std::vector<std::string> props = m.properties<vertex_descriptor>();
  BOOST_FOREACH(std::string p, props){
    std::cout << p << std::endl;
  }
  
  // delete the string property again
  m.remove_property_map(name);
  return 0;
}
```



