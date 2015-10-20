---
layout:     post
title:      Post Headline
date:       2015-10-18 11:35:00
author:     Darren Maguire
tags: 		result
---
<!-- Start Writing Below in Markdown -->


**Crystal Plasticity Codes**
---

#Summary

When modeling polycrystalline material using crystal plasticity UMAT with Abaqus an input microstructure needs to be created. Dream 3D is an open source tool to generate synthetic microstructures [1].

However, Dream3D gives the Euler angles but not the node and element lists, which can be directly inputted into abaqus. Since Abaqus has certain numbering pattern used to create elements from nodes, some special tools (e.g.Hypermesh) have to be used to generate mesh files or it has to be scripted [2]. But Dream3D has its own numbering pattern for voxels. The main goal of this project is to create a script that will match the numbering between Abaqus and Dream3D.  

Matlab was chosen to create the script that will generate the node and element lists. The structure is a cubic structure with the same number of elements on each side ( n by n by n). To apply simple boundary conditions,  node sets are needed for each face of the cube. 
The first matlab code, shown below, gives the code that creates the node text file, which designates points in 3D space in abacus. The next code creates the element text file, which designates each cubic element from 8 nodes. The two text files are entered into an input file for Abaqus, which then creates the cubic structure. The numbering patterns in Abacus and Dream3D are shown below, as are the codes.  

---

**Here is an example of the text files for a cube of dimension 2:**

> Node Text File

The node is the first number in each column, and it is followed by the x,y, and z coordinates. Abaqus requires a period after each coordinate, and a comma after the first three numbers in each row. 
![Node file](https://lh3.googleusercontent.com/-9U7giWxTGkM/ViZf_i-RXsI/AAAAAAAAAAg/G1f4_SehT8g/s1000/Presentation1a.png "Presentation1a.png")

> Example of Nodes

They are numbered starting from 0,0,0 and move along the x-axis. They then move up the y axis until the z = 0 plane is filled, and then they move from z = 0 to the z = n plane. Nodes are represented by red dots with black numbers, and the elements are represented by blue numbers.
![enter image description here](https://lh3.googleusercontent.com/-WwWNoCln90g/ViaQ_U5gZmI/AAAAAAAAACU/WltFvEAtOZQ/s1000/Presentation15.jpg "Presentation15.jpg")

---

> Element Text File

The element is the first number in each column, and it is followed by the eight nodes that create it. 
![Element file](https://lh3.googleusercontent.com/-vnEUcW5UTaw/ViZ-BNTNB1I/AAAAAAAAABA/yNcNZ1N8FOY/s1000/Presentation1.jpg "Presentation1.jpg")

> Example of Element
The picture shows how element 1 would be created for the 2 by 2 by 2 cube. This numbering pattern holds true for all of the elements.
![enter image description here](https://lh3.googleusercontent.com/-howPvQq22nc/ViaO7SnYyXI/AAAAAAAAACA/O8wwMLxf8j8/s1000/Presentation14.jpg "Presentation14.jpg")

---
> Node Set Text File

The below format is required for Abaqus, and it gives the nodes on each of the six faces.
![enter image description here](https://lh3.googleusercontent.com/-tzUHS91TNNo/ViZ-fWXrOLI/AAAAAAAAABM/xiTX2YSrEug/s1000/presz.jpg "presz.jpg")

---

**Code for Node file:**

The two inputs are in, an integer that gives the dimension of the cube, and node, a string that is the name of the text file.
 
	function [] = nodefile(in,node)
	 x = (in+1)^3;
	 y = (in+1)^2;
	 z = (in+1);     
	 cell_array = cell(x,4);      
     for i = 1:x
        cell_array(i,1) = {num2str(i)};
     end
	 t = 1;
     for i = 1:y
        j = 0;
        while j < z
        cell_array(t,2) = {num2str(j)};
        j = j+1;
        t = t +1;
        end
     end
    
     d = 1;
     j = 0;
     for i = 1:y
        j = j+1;
        if j > z
            j = 1;
        end
        t = 0;
        while t < z 
        cell_array(d,3) = {num2str(j-1)};
        t = t +1 ;
        d = d + 1;
        end
     end

     j = 0;
     d = 1;
     for i = 1:z
        t = 0;
        j = j+1;
        while t < y 
            cell_array(d,4) = {num2str(j-1)};
            t = t+1;
            d = d+1;
        end
     end

	 testfile = cell_array;

     for i = 1:x
        for j = 2:3
            lol = testfile{i,j};
            lol(end+1) = '.';
            lol(end+1) = ',';
            testfile{i,j} = lol;
        end
      end
    
     for i = 1:x
        help = testfile{i,4};
        help(end+1) = '.';
        testfile{i,4} = help;
     end
    
     for i = 1:x
        not = testfile{i,1};
        not(end+1) = ',';
        testfile{i,1} = not;
     end

	 fileID = fopen(node,'w');

     for i = 1:x
        for j = 1:4
        fprintf(fileID,'%s',testfile{i,j});
        end
        fprintf(fileID,'\n',testfile{i,:});
     end
	end

**Code for Element file:**
The two inputs are in, an integer that gives the dimension of the cube, and element, a string that is the name of the text file.
		
	function [] = element(in,element)
	x = ((in+1)^2)+in+2;
	y = (in^2);
	z = (in^3);
	cell_array = cell(y,9);
	for i = 1:z
	    cell_array{i,1} = i;
	end
    cell_array{1,2} = x;
    cell_array{1,3} = x-(in+1);
    cell_array{1,4} = 1;
    cell_array{1,5} = 2 + in;
    cell_array{1,6} = x + 1;
    cell_array{1,7} = x - in;
    cell_array{1,8} = 2;
    cell_array{1,9} = 3+in;
	a = 1;
	b = 2;
	    for f = 2:9
	        for i = 1:(in-1)
	            cell_array{i+1,f} = cell_array{i,f} + a;
	        end
		    for i = 1:in:(in^3)
		      cell_array{i+in,f}=cell_array{i+in-1,f}+b;
	            for j = 1:(in-1)
	                j = j-1;                
	cell_array{i+in+1+j,f} = cell_array{i+in+j,f} + 1;
	            end
	        end
	        for j = 1:(in-1)
	            cst2 = 0;
	            for i = 1:(in^2)
	                cst = (1+in)*j;
	                cst2 = ((in^2)+i)+((j-1)*in^2);
	      cell_array{cst2,f} = cell_array{cst2,f} + cst;
	            end
	        end
	    end
    
    x = cell_array(1:z,1:9);
    
	for i = 1:(in^3)
	    for j = 1:9
	        x{i,j} = num2str(x{i,j});
	    end
	end
	
		for i = 1:(in^3)
	    for j = 1:8
	        lol = x{i,j};
	        lol(end+1) = ',';
	        x{i,j} = lol;
	    end
	end

	fileID = fopen(element,'w');

    for i = 1:z
    for j = 1:9
        fprintf(fileID,'%s',x{i,j});
    end
        fprintf(fileID,'\n',x{i,:});
    end

**Node Set Code**
The two inputs are in, an integer that gives the dimension of the cube, and set, a string that is the name of the text file.
		
		function [] = nodeset(in,set)
		x0 = in+1;
	    x1 = (in+1)^2;
	    x2 = ((in+1)^3 - (in+1)^2 +1);
	    x3 = (in+1)^3;
	    k1 = 1:x1;
	    k2 = x2:x3;
	    
    main_vec = cell(6,x1);
    
    %XY In CELL
    for i = 1:2
            for j = 1:x1
                if i == 1
                main_vec{i,j} = k1(j);
                elseif i == 2
                main_vec{i,j} = k2(j);
                end
            end
    end
    
    %XZ in vector
    xz = 1:(in+1);
    vecxz1 = [];
        for j = 1:x0
            keep = xz+((in+1)^2)*(j-1);
            vecxz1 = [ vecxz1 keep];  
        end
	for i = 1:x1
	    main_vec{3,i} = vecxz1(i);
	end
	    xz2 = (in+1)^2 - in;
	    helpme = xz2:x1;
	    vecxz2 = [];
	        for j = 1:x0
	            keep = helpme+((in+1)^2)*(j-1);
	            vecxz2 = [ vecxz2 keep];
	        end     
	    for i = 1:x1
	        main_vec{4,i} = vecxz2(i);
	    end   
	        
    %%YZ in vector
    yz = 1:x0:(x1-in);
    vecyz = [];
        for j = 1:x0
            keep = yz+((in+1)^2)*(j-1);
            vecyz = [ vecyz keep];  
        end
        
        for i = 1:x1
            main_vec{5,i} = vecyz(i);
        end
        
     yz2 = x0:x0:x1;
     vecyz2 = [];
        for j = 1:x0
            keep = yz2+((in+1)^2)*(j-1);
            vecyz2 = [ vecyz2 keep];
        end
        
        for i = 1:x1
            main_vec{6,i} = vecyz2(i);
        end
    
	 fileID = fopen(set,'w');


    for i = 1:6
        for j = 1:x1
	        main_vec{i,j} = num2str(main_vec{i,j});
	        lol = main_vec{i,j};
	        lol(end+1) = ',';
	        main_vec{i,j} = lol;
        end
    end

    for i = 1:6
        if i == 1 || i == 2
	         fprintf(fileID,'*Nset, nset= xy%d',i);
	         fprintf(fileID,'\n');
        elseif i == 3 || i == 4
	         fprintf(fileID,'*Nset, nset= xz%d',i-2);
	         fprintf(fileID,'\n');
        elseif i == 5 || i == 6
	         fprintf(fileID,'*Nset, nset= yz%d',i-4);
	         fprintf(fileID,'\n');
        end
         yy = 0;
         counter = 0;
        for j = 1:x1
	        x = main_vec{i,j};
	        fprintf(fileID,'%s',x);
	        yy = yy+1;
	        counter = counter+1;
		        if yy == 12
		            fprintf(fileID,'\n');
		            yy = 0;
		        end
		        if counter == x1
		            fprintf(fileID,'\n**\n');
		        end
	        end
        
	    end
	end

---


#Links:

[More on DREAM3D][1]
[More on Abaqus][2]

[1]:http://dream3d.bluequartz.net/?page_id=71
[2]:http://www.3ds.com/products-services/simulia/products/abaqus/
	



