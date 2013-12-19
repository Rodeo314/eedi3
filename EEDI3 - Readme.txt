
v0.9.1 - July 23, 2010

Haven't had time to make a good help file so this is pretty rough/short.

   ** Note: eedi3 is threaded using openmp, and is compiled with visual studio 2005.
   ** Therefore, it needs the visual studio 2005 redistributable package to be
   ** installed (for vcomp.dll).


Info:

     eedi3 works by finding the best non-decreasing (non-crossing) warping between two lines by minimizing a cost functional.
   The cost is based on neighborhood similarity (favor connecting regions that look similar), the vertical difference created
   by the interpolated values (favor small differences), the interpolation directions (favor short connections vs long), and
   the change in interpolation direction from pixel to pixel (favor small changes).


Functions:


  "eedi3", "c[field]i[dh]b[Y]b[U]b[V]b[alpha]f[beta]f[gamma]f[nrad]i[mdis]i[hp]b[ucubic]b[cost3]b[vcheck]i[vthresh0]f[vthresh1]f[vthresh2]f[sclip]c[threads]i"

  "eedi3_rpow2", "c[rfactor]i[alpha]f[beta]f[gamma]f[nrad]i[mdis]i[hp]b[ucubic]b[cost3]b[vcheck]i[vthresh0]f[vthresh1]f[vthresh2]f[cshift]s[fwidth]i[fheight]i[ep0]f[ep1]f[threads]i"


Parameters:

   field/dh/y/u/v are the same as nnedi2/nnedi3. 

   rfactor/cshift/fwidth/fheight/ep0/ep1 are the same as nnedi2_rpow2/nnedi3_rpow2. 


   alpha/beta/gamma (defaults: 0.2,0.25,20.0):

     These trade off line/edge connection vs artifacts created. alpha and beta must be in the range [0,1], and the sum
     alpha+beta must be in the range [0,1]. alpha is the weight given to connecting similar neighborhoods.. the larger
     it is the more lines/edges should be connected. beta is the weight given to vertical difference created by the
     interpolation... the larger beta is the less edges/lines will be connected (at 1.0 you get no edge directedness at all). 
     The remaining weight (1.0-alpha-beta) is given to interpolation direction (large directions (away from vertical) 
     cost more)... so the more weight you have here the more shorter connections will be favored. Finally, gamma penalizes
     changes in interpolation direction, the larger gamma is the smoother the interpolation field between two lines 
     (range is [0,inf].

     If lines aren't getting connected then increase alpha and maybe decrease beta/gamma. Go the other way if you are 
     getting unwanted artifacts.


   nrad/mdis (defaults: 2,20):

     nrad sets the radius used for computing neighborhood similarity. Valid range is [0,3]. mdis sets the maximum
     connection radius. Valid range is [1,40]. If mdis=20, then when interpolating pixel (50,10) (x,y), the farthest
     connections allowed would be between (30,9)/(70,11) and (70,9)/(30,11). Larger mdis will allow connecting lines
     of smaller slope, but also increases the chance of artifacts. Larger mdis will be slower. Larger nrad will be
     slower.


   hp/ucubic/cost3 (defaults: false, true, true):

      These are speed vs quality options. hp=true, use half pel steps, hp=false, use full pel steps. ucubic=true, use 
      cubic 4 point interpolation, ucubic=false, use 2 point linear interpolation. cost3=true, use 3 neighborhood cost
      function to define similarity, cost3=false, use 1 neighborhood cost function.


   vcheck/vthresh0/vthresh1/vthresh2/sclip (defaults: 2, 32.0, 64.0, 4.0, NULL):

      vcheck settings:

          0 - no reliability check
          1 - weak reliability check
          2 - med reliability check
          3 - strong reliability check

      If vcheck is greater than 0, then the resulting interpolation is checked for reliability/consistency. Assume
      we interpolated pixel 'fh' below using dir=4 (i.e. averaging pixels bl and cd).

           aa ab ac ad ae af ag ah ai aj ak al am an ao ap
                                eh          el
           ba bb bc bd be bf bg bh bi bj bk bl bm bn bo bp
                    fd          fh          fl
           ca cb cc cd ce cf cg ch ci cj ck cl cm cn co cp
                    gd          gh
           da db dc dd de df dg dh di dj dk dl dm dn do dp

      When checking pixel 'fh' the following is computed:

            d0 = abs((el+fd)/2 - bh)
            d1 = abs((fl+gd)/2 - ch)

            q2 = abs(bh-fh)+abs(ch-fh)
            q3 = abs(el-bl)+abs(fl-bl)
            q4 = abs(fd-cd)+abs(gd-cd)

            d2 = abs(q2-q3)
            d3 = abs(q2-q4)

            mdiff0 = vcheck == 1 ? min(d0,d1) : vcheck == 2 ? ((d0+d1+1)>>1) : max(d0,d1)
            mdiff1 = vcheck == 1 ? min(d2,d3) : vcheck == 2 ? ((d2+d3+1)>>1) : max(d2,d3)

            a0 = mdiff0/vthresh0;
            a1 = mdiff1/vthresh1;
            a2 = max((vthresh2-abs(dir))/vthresh2,0.0f)
						
            a = min(max(max(a0,a1),a2),1.0f)
						
            final_value = (1.0-a)*fh + a*cint


        ** If sclip is supplied, cint is the corresponding value from sclip. If sclip isn't supplied,
           then vertical cubic interpolation is used to create it.


   threads (default: 0):

      Sets the number of threads used by openmp.

         0 = default (environment variable OMP_NUM_THREADS)
         > 0 = calls omp_set_num_threads(threads)


Changes:

    v0.9.1

       - fix field=0/1 flipped with rgb24 input
       - fix possible reading off the edge of a frame with cost3=true

