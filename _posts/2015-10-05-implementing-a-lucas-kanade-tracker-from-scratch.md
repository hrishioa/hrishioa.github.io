---
layout: post
title: "Implementing a Lucas-Kanade tracker from Scratch"
date: 2015-10-05 22:11:19
image: '/assets/img/blog-image.png'
description: 'Understanding the basics of Optical Flow and XCode'
tags: 
- Lucas-Kanade
- Optical Flow
- C++
- XCode
categories:
twitter_text:
---

So I got an assignment a while ago to implement a [Lucas-Kanade Tracker][lktwiki] from scratch.

Text goes here:

Sample code to test formatting:

{% highlight C++ %}
    Mat lucasKanade(Mat imgA, Mat imgB, int xsarea, int ysarea, int xarea, int yarea, deque<Point> corners, bool verbose=true, char filename[] = NULL)
    {
        Mat outimg = imgB.clone();
        
        int dimx = imgA.cols, dimy = imgA.rows;
        
        //iterate through each corner to find flow vectors
        for(int i=0;i<corners.size();i++)
        {
            Point cur_corner = corners[i];
            Point corner_mid = Point(corners[i].x+(int)(xarea/2),cur_corner.y+(int)(yarea/2));
            
            //Draw the corner in the out-image
            rectangle(outimg, cur_corner, cur_corner+Point(xarea,yarea), Scalar(0), 2);
            
            //Set range parameters
            Range sry = Range(max(0,corner_mid.y-(int)(ysarea/2)), min(dimy,corner_mid.y+(int)(ysarea/2)));
            Range srx = Range(max(0,corner_mid.x-(int)(xsarea/2)), min(dimx,corner_mid.x+(int)(xsarea/2)));
            
            //Now that we've found search windows, we can proceed to calculate Ix and Iy for each pixel in the search window
            //Ix
            Mat Ix = imgA.clone(), Iy=imgA.clone();
            
            int range=1;
            double G[2][2] = {0,0,0,0};
            double b[2] = {0,0};
            
            for(int x=srx.start; x<=srx.end; x++)
                for(int y=sry.start;y<=sry.end;y++)
                {
                    int px=x-range, nx=x+range;
                    int py=y-range, ny=y+range;
                    if(x==0) px=x;
                    if(x>=(dimx-1)) nx=x;
                    if(y==0) py=0;
                    if(y>=(dimy-1)) ny=y;
  
                    double curIx = ((int)imgA.at<uchar>(y,px)-(int)imgA.at<uchar>(y,nx))/2;
                    double curIy = ((int)imgA.at<uchar>(py,x)-(int)imgA.at<uchar>(ny,x))/2;
                    Ix.at<uchar>(y,x) = curIx;
                    Iy.at<uchar>(y,x) = curIy;
                    
                    //calculate G and b
                    G[0][0] += curIx*curIx;
                    G[0][1] += curIx*curIy;
                    G[1][0] += curIx*curIy;
                    G[1][1] += curIy*curIy;
                    
                    double curdI = ((int)imgA.at<uchar>(y,x)-(int)imgB.at<uchar>(y,x));
                    
                    b[0] += curdI*curIx;
                    b[1] += curdI*curIy;
                }
            
            //Since it's a royal pain-in-the-ass to download and link matrix libraries to my XCode
            //for just a single operation, we're just gonna do it by hand
            double detG = (G[0][0]*G[1][1])-(G[0][1]*G[1][0]);
            double Ginv[2][2] = {0,0,0,0};
            Ginv[0][0] = G[1][1]/detG;
            Ginv[0][1] = -G[0][1]/detG;
            Ginv[1][0] = -G[1][0]/detG;
            Ginv[1][1] = G[0][0]/detG;
            
            double V[2] = {Ginv[0][0]*b[0]+Ginv[0][1]*b[1],Ginv[1][0]*b[0]+Ginv[1][1]*b[1]};
            if(verbose)
                printf("\nFor the corner (%d,%d) - v = (%f,%f)",cur_corner.x,cur_corner.y,V[0],V[1]);
            
            line(outimg, corner_mid, corner_mid+Point(V[0]*10,V[1]*10), Scalar(1), 1);
        }
        string winCorImg = "Corners found";
        namedWindow(winCorImg, WINDOW_AUTOSIZE);
        imshow(winCorImg, outimg);
        waitKey(0);
        if(filename!=NULL)
        {
            cout << string("\nWriting lkoutput to o") << string(filename);
            imwrite(string("o")+string(filename), outimg);
        }
        return outimg;
    }
{% endhighlight %}

Here's some more text. Here's some `emphasized text` to make sure it works.

##OCaml Code

Here's a test to make sure OCaml highlighting is good.

{% highlight ocaml %}
(*Implementation of the fold_left, for consistency*)
let rec fold_left f list init =
  match list with
      [] -> init
    | h::t -> fold_left f t (f h init);;

(*Fold_right function*)
let fold_right func =
  fun list init -> fold_left (fun v acc -> (fun x ->  acc (func v x))) list (fun x -> x) init;;

{% endhighlight %}

Once this code is shown, we can discuss it below using function markers. Of course, the important function here is `fol_right`, but the others are included.

[lktwiki]:https://en.wikipedia.org/wiki/Lucas%E2%80%93Kanade_method