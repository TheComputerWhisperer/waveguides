# waveguides, code for certian gui

```python 

# -*- coding: utf-8 -*-
"""
Created on Tue Nov 17 15:49:52 2015

@author: physicsuser
"""
#doesnt work for zero 
from sympy.solvers import solve
from sympy import Symbol
def nsub():
    
    ti = input('Enter the waveguide core thickness in microns:')
    tf = float(ti)
    if tf < 0.001:
        print('The waveguide is too thin.')
        nsub()
        
    t=(tf*1e-6 )
    x = input('\n' + 'Enter x for your' + ' Al_(x)Ga_(1-x)As' +  ' waveguide core Aluminium fraction:')
    xi=float(x)    
    if xi >1.0:
        print('\n' + 'x must be less than unity.')
        x = input('\n' + 'Enter x for your' + ' Al_(x)Ga_(1-x)As' +  ' waveguide core Aluminium fraction:')
        xi=float(x) 
        if xi >1.0:
            print('\n' + 'x must be less than unity, once more and I will send you ALL the way back to the beginning.')
            x = input('\n' + 'Enter x for your' + ' Al_(x)Ga_(1-x)As' +  ' waveguide core Aluminium fraction:')
            xi=float(x)
            if xi >1.0:
                xi = 0#reset maybe?
                nsub()#three strikes
    ni = 3.3 - (0.53*xi) + (0.09*(xi**2)) #+ 0.29 to align with sell in x=0.4 to x=0.7
    wavelength = 780e-9
    constant = (wavelength**2)/32
    t = 1e-6#(tcof_slider.get())*(10**(texp_slider.get()))  
    deln = constant / (ni*(t**2)) 
    n3 = deln + ni
    z = Symbol('z')
    yfrac = ((3.3 - (0.53*z) + (0.09*z**2)) - n3)
    solutions = solve (yfrac,z)
    for i in solutions:
        if i < 1.0: 
            print ('\n' + 'For x = %.5g in the core, the miniumum y for the substrate = %.5g' % (xi,i))#check the physics behind x and y
nsub()


'''The gui for n and band gap as a function of Aluminium concentration x.  x is on a slider and you have two execute buttons
 which when pressed will print the corresponding answer on the gui and relevant information associated with the answwer.'''

from tkinter import *
import math as mh
from sympy.solvers import solve
from sympy import Symbol

'''This function finds the band gap using the equations from 
http://www.ioffe.ru/SVA/NSM/Semicond/AlGaAs/bandstr.html it also evaluates whether you are in
the direct or indirect band gap region.  No attempts have been made to incorporate the 
physics of electron phonon interactions 'energetically' near to where the direct to indirect 
crossover occurs.  The transition is defined by the solution to the two
quadratic equations (vbr representing valence band to Gamma valley (direct) and vbx representing 
valence band to X valley (indirect) so it should only serve as a guide.  '''

def BandGap(): 
    #This block is for solving the cross over point
    z = Symbol('z')
    direct_minus_indirect = (1.424 + 1.155*z + 0.37*z*z - (1.9 + 0.124*z + 0.144*z*z))
    solutions = solve (direct_minus_indirect,z)
    crossover_x = (solutions[1]) #I have checked, the physical solution we need is in solutions[1]
    
    x = (x_slider.get()) # this fetches the x value from the slider, defined in the GUI code below
    vbx= 1.9 + (0.124*x) + (0.144*x**2)
    vbr= 1.424 + 1.155*x + 0.37*x*x
    
    '''The aluminium concentration that produces a direct semiconductor    
    with a band gap that corresponds 780 nm is approx. x = 0.1371 which is in b[1],
    future improvements would involve returning a corresponding absorption coefficient evaluated for that
    x value, ie putting more physics into the conditionals below.  The figures used here are arbitrary. '''
    
    ex = Symbol('ex')
    a = 1.424 + 1.155*ex + 0.37*ex**2 - (1240/780.2)
    bg = solve(a) 
    bgap780 = (bg[1]) 
    
   
    if x > crossover_x:
        bandgap.set('Egap(%.3g) = %.3g eV, Indirect bandgap, larger than Egap(780nm).' % (x,vbx)) #bandgap is defined in the GUI set up
    else:
        if x <= bgap780:
            bandgap.set('Egap(%.3g) = %.3g eV, Direct bandgap, highly absorbing at 780nm.' % (x,vbr))
        else:
            bandgap.set('Egap(%.3g) = %.3g eV, Direct bandgap, larger than Egap(780nm).' % (x,vbr))

'''This function finds the refractive index as a fraction of x using the Sellmeier equation from 
p67 in Hunsperger's 'Integrated optics' ref[50,51]. The variable wavelength_mu can be changed but must be stated
in microns. '''    

def n():
    x = (x_slider.get()) # this fetches the x value from the slider, defined in the GUI code below
    wavelength_mu = 0.7802
    A= 10.906-(2.92*x)
    B= 0.97501
    CsX= (0.52886 - 0.735*x)**2
    CbX= (0.30386-0.105*x)**2
    D= ((0.002467*1.41*x) + 0.002467)
    ysq= wavelength_mu**2
    if x <= 0.3600000000010:
        
        ans= mh.sqrt(A + (B*(1/(ysq-CsX))) - (D*(ysq)))
    else: 
        ans = mh.sqrt(A + (B*(1/(ysq-CbX))) - (D*(ysq)))
    refInd.set('n(%.3g) = %.3g, for λ = 780 nm.' % (x,ans)) #refInd is defined in the GUI set up

'''Gui set up '''   

root = Tk()

# Our state variables for the Gui#
bandgap = StringVar() 
refInd = StringVar() 

#title
lbl = Label(root,text='AlGaAs: refreactive index and bandgap with varying x')
lbl.grid(row=0, column=0, columnspan=2)

#instruction for the gui user
from_lbl = Label(root, text='Enter aluminium fraction: x on slide bar')
from_lbl.grid(row=3, column=0, columnspan=3)

#buttons to execute function
convert_btn = Button(root, text='Eg(x)', command=BandGap)
convert_btn.grid(row=1, column=0)

convert_btn = Button(root, text='n(x) ', command=n)
convert_btn.grid(row=2, column=0)

#results## Note the str variables
result_lbl = Label(root,textvariable = bandgap)
result_lbl.grid(row=1, column=1)

result_lbl = Label(root,textvariable = refInd)
result_lbl.grid(row=2, column=1)



#slider bar
x_slider = Scale(root,from_=0,to=1,tickinterval=0.10,length=600, resolution = 0.005, orient=HORIZONTAL)
x_slider.grid(row=4, column=0, columnspan=2)

root.mainloop()
```

