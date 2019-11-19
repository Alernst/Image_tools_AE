/*ClearOutside_3D.ijm
*This ImageJ- macro is designed to isolate a 3D volume from fluorescence multicolor image stacks. 
 * 
* Principle:
* You draw polygon selections around the target structure in a subset of Z-planes and add them to the roimanager. 
* To reduce the workload, the rois for the rest of the Z-planes are linearly interpolated. Subsequently, all pixels outside of the rois 
* are cleared/set to zero. This is applied to all channels in the hyperstack.
* 
* Application: 
* In model organisms like the zebrafish fluorescent markers are often not unique for the target structure   
* or you would like to extract just part of a structure. 
* We are working with embryonic zebrafish hearts and the typical marker for the heart in immunostainings labels 
* all muscle cells. To quantify the volume or cellnumbers of the heart, we developed this tool.
* 
* Alexander Ernst
* 
* ImageJ macro language, tested on FIJI with ImageJ 1.52p Java 1.8.9_211
*/

//Empty roimanager and close all images 
roiManager("reset");
close("*");

//Open a dialog to load the images file
run("Open...");
titleMain=getTitle()

//Read metadata
getDimensions(width, height, channels, slices, frames);

//Check whether the movie is a time lapse, it won't work for 4D XYZT Hyperstacks. I am currently working on it. 
if(frames < 2)
{
	//A dialog appears to define which channel is used as reference structure to isolate the heart
	Dialog.create("3D volume extraction by linear interpolation");
	Dialog.addNumber("In which channel is your target structure?", 1);
	Dialog.show();
	Channel=Dialog.getNumber();
	run("Duplicate...", "duplicate title=Target channels="+ Channel +"");
	selectWindow("Target");
	roiManager("reset");

	//These steps are needed to crop the Z-dimension, it reduces the number of selections that need to be drawn and ensures proper function. 
	setTool("rectangle");
	waitForUser("To crop the Z-dimension, please make a rectangular selection 'add'(shortcut press 'T') in which your structure is visible to the Roi manager");
	selectWindow("Target");
	roiManager("select", 0);
	slN1=getSliceNumber();
	roiManager("select", 1);
	slN2=getSliceNumber();
	roiManager("reset");
	run("Select None");
	roiManager("Deselect");
	run("Duplicate...", "duplicate title=Target_crop range="+slN1+"-"+slN2+" use");
	selectWindow("Target");
	close();
	selectWindow("Target_crop");

	// Create rois manually in a subset of Z-planes and subsequently interpolate them linearly. Just one per plane 
	waitForUser("Create selections around your target structure in every 5-10th Z-plane and add each of them to the roimanager.");
	roiManager("Deselect");
	roiManager("Sort");
	roiManager("Interpolate ROIs");
	
	n=roiManager("count");
	
	// Duplicate original cropped in the Z-dimension 
	selectWindow(""+titleMain+"");

	run("Duplicate...", "duplicate title=Isolated_CH slices="+ slN1 +"-"+ slN2 +"");
	selectWindow(""+titleMain+"");
	close();
	selectWindow("Isolated_CH");
	//Apply structure isolation to all channels
	for(i=1;i<channels+1; i++)
		{
		Stack.setChannel(i);
		
		for(j=0;j<n;j++)
		{
			roiManager("Select", j);
			run("Clear Outside","slice");
		}
	}
}
// Will appear in the case of 4D time-lapse data
else {
	showMessage("does not work yet on time-lapse");
}	
close("Target_crop");
