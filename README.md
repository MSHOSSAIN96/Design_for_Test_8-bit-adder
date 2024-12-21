**Chapter 1 Introduction**

**1.1. Goals**

The goal of this directory is to learn design for test and generation of test patterns. In this part of the lab, I will use a simple adder design for better understanding of the fellow without dealing with design complications. In more complex designs, some further changes to the design might be necessary. In this experiment, I will modify the architecture by adding some scan input and output pins.

In this directroy, I am going to exersice the following steps:

**a.Modify the design**

**b.Integrate the scan architechture into the design**

**c.Automatic Test Pattern Generation (ATPG)**

**d.Simulating the design using the generated patterns**

**1.2. Orginize your data and setup the tool**

Before I start, it is a good idea to orginize my data by creating set of directories where I am going to put the data generated during the design flow. Go to the directory of **dft/do_synth** and create the following directories to organize the date:

cd dft/do_synth

mkdir results

mkdir reports

mkdir cmd

mkdir log

**Chapter 2 Design the unit (RTL)**

The first step in the design flow is to create the RTL code of the adder using VHDL. I just need a text editor to write VHDL. In the diectory, go to the directory dft/rtl and open a text editor to create the file my_adder.vhd. There are a lot of possibilities, as vim, nano, gedit, etc. I will use emacs.

In the terminal execute:

cd dft/rtl

and then,

emacs my_adder.vhd &

The code can be as follows:

library ieee;

use ieee.std_logic_1164.all;

use ieee.numeric_std.all;

entity my_adder is

  port (
    clk,rstn: in std_logic;
    
    dta,dtb : in  std_logic_vector(7 downto 0);
    
    dto : out std_logic_vector(7 downto 0));
    
end my_adder;


architecture rtl of my_adder is

  signal dta_int,dtb_int,dto_int : std_logic_vector(7 downto 0);

begin  -- architecture rtl

  -- Define the memory elements
  
  process (clk, rstn) is
  
  begin  -- process
  
    if rstn = '0' then                   -- asynchronous reset (active low)
      
       dta_int <= (others=>'0');
       
       dtb_int <= (others=>'0');
       
       dto     <= (others=>'0');
       
    elsif clk'event and clk = '1' then  -- rising clock edge
    
       dta_int <= dta;
       
       dtb_int <= dtb;
       
       dto     <= dto_int;
       
    end if;
    
  end process;
  

  -- Define the adder
  
  dto_int <= std_logic_vector( signed(dta_int)+signed(dtb_int) );
  

end architecture rtl;

In order to integrate the scan chain into the design, I need to modify the design. Define SERIAL_IN and SCAN_EN as input ports of form std_logic and SERIAL_OUT as output port of the std_logic. Save the changes and close the text editor.

SERIAL_IN, SCAN_EN: in std_logic;

SERIAL_OUT: out std_logic

**Chapter 3 Lunch design compiler**

For this lab I am going to use the Design Compiler tool from Synopsys as well as INNOVUS from Cadence. In the terminal, go to the directory dft/do_synth. for using the Design Compiler create a sourceme.sh file to do the setup.

export SNPSLMD_LICENSE_FILE=28231@item0096

export PATH=/usrf01/prog/synopsys/syn/R-2020.09-SP4/bin:${PATH}

Now I can source that file.

source sourceme.sh

Before open the synthesis tool, it is practical to select the library of standard cells that I use and to instruct the tool to save the log files into the directories that I have previously defined. I can do that creating a .synopsys_dc.setup file. Run emacs .synopsys_dc.setup.

First, I can add the following commands to .synopsys_dc.setup file; they instruct the tool to use directories.

define_design_lib work -path ./tool/work

set_app_var  view_log_file        ./log/synth_view.log

set_app_var  sh_command_log_file  ./log/synth_sh.log

set_app_var  filename_log_file    ./log/synth_file.log

set_app_var search_path       [concat ./cmd/  [get_app_var search_path] ]    

And afterwards you can write the command to define the library to use.

set library_path "../../../0_FreePDK45/LIB/"

set library_name "NangateOpenCellLibrary_typical_ccs_scan.db"

set_app_var target_library    $library_name

set_app_var link_library      [concat $library_name dw_foundation.sldb "*"]

set_app_var search_path       [concat $library_path [get_app_var search_path] ]

set_app_var synthetic_library [list dw_foundation.sldb]

set_app_var symbol_library    [list class.sdb]

set_app_var vhdlout_use_packages { ieee.std_logic_1164.all  NangateOpenCellLibrary.Components.all }

set_app_var vhdlout_write_components FALSE

Save this file. Now I can start the tool. Launch design_vision in the terminal. To save the log in a directory, I can add a linux pipe tee log/synthesis.log that redirects the log output to a file called log/synthesis.log.

design_vision | tee log/synthesis.log &

![Screenshot 2024-12-21 205200](https://github.com/user-attachments/assets/7b3cfd89-0496-4f2b-8492-317544da7940)


**4. DFT configuration and synthesis**

**4.1. Design import**

The first step is to read the RTL code. In the menu select File → Read . In the new window select the VHDL code to read, i.e., ../rtl/my_adder.vhd and click Open.

![Screenshot 2024-12-21 205416](https://github.com/user-attachments/assets/fcb5e7ef-597f-4a3e-b6c8-b7912354310d)

Alternatively, I can type the following command line in the design_vision prompt.

read_file -format vhdl {../rtl/my_adder.vhd}

The tool reads and "understands" the code. In the log file I can see the elements that it has inferred. For example, it found the following registers:

![Screenshot 2024-12-21 205540](https://github.com/user-attachments/assets/22f16cf9-2f41-4603-8e09-018d4c5b9f9c)


**4.2. DFT synthesis**

Before configure the DFT once compile the design and have a look at the schematic of design. I can type the following command line in the design_vision prompt.

compile

Observe that the DFFR_X1 registers are used by the synthesis tool.

Now, I am going to configure the DFT. There are many options which should be set to configure the DFT. Among all options, I set the clock, test period, type of scan cells, the reset, scan input and output, Scan enable and test mode. There are three widly used scan cells, Muxed-D Scan Cell, Clocked Scan Cell and LSSD Scan Cell. Here, I use MUXED-D scan cells.

set_scan_configuration -style multiplexed_flip_flop

set test_default_period 100

set_dft_signal -view existing_dft -type ScanClock -timing {45 55} -port clk

set_dft_signal -view existing_dft -type Reset -active_state 0 -port rstn

set_dft_signal -view spec -type ScanDataIn -port SERIAL_IN

set_dft_signal -view spec -type ScanDataOut -port SERIAL_OUT

set_dft_signal -view spec -type ScanEnable -port SCAN_EN -active_state 1

create_test_protocol

Now I can begin the DFT synthesis. Execute the following commands:

compile -scan

Using this command, I redo the compilation by replacing all the sequential elements by scan equivalent. Observe the replacement of the sequential elements of the DFFR_X1 with the scan equivalent of SDFFRS_X1 in design.

I can preview the scanned design for scan chain information using:

preview_dft

Besides, I can check the design for design rule violations using the following command:

dft_drc

After that I can specify the scan chain. I can select the number and length of the scan chains. I define only one scan chain for this design. In addition, I define that no different clock edges may occur in the scan chain. The input and output of the scan chain is also specified.

set_scan_configuration -chain_count 1

set_scan_configuration -clock_mixing no_mix

set_scan_path chain1 -scan_data_in SERIAL_IN -scan_data_out SERIAL_OUT

Then, the scan chain can be inserted into the design. In addition, the command set_scan_state_scan_existing indicates that the scan chain is fully implemented or not.

insert_dft

set_scan_state scan_existing

Check the schematic of design once more. The scan chain inserted in the design. Finally, I can produce some reports including the scan chain and the instances in each chain using the report_scan_path as follows:

report_area

report_timing

report_power 

report_scan_path -view existing_dft -chain all > reports/chain.rep

report_scan_path -view existing_dft -cell all > reports/cell.rep

After top-level scan insertion, I need different files for test pattern generation with TetraMax from synopsys.

Write out the test protocol file in Standard Test Interface Language (STIL) format and write out a Verilog or VHDL top-level netlist. For example, use the following tcl commands to save the necessary files as follows:

change_names -hierarchy -rule verilog

write -format verilog -hierarchy -out results/my_adder_dft.vg

write -format ddc -hierarchy -output results/my_adder_dft.ddc

write_scan_def -output results/my_adder_scan.def

set test_stil_netlist_format verilog

write_test_protocol -output results/my_adder.stil

Using the above commands, I will write a gate level netlist .vg. set test_stil_netlist_format verilog command identifies the netlist format that I am exporting to TetraMAX ATPG. I also write out the test protocol file using the write_test_protocol command.

**5. ATPG**

**5.1. Lunch TetraMAX**

In this part, I want to use the TetraMax from synopsys to create the test patterns. Redirect to the tetramex folder cd dft/tetramx and start the software as follows:

export PATH="/usrf01/prog/synopsys/txs/R-2020.09-SP4/bin:${PATH}"

tmax

![Screenshot 2024-12-21 211432](https://github.com/user-attachments/assets/aa867a42-3947-4362-a8b7-95368aef2efb)

**5.2. Read the netlist and library files**

In this step, I read the gate level netlist of the design as well as the library definition of each cell in verilog format. Browse the netlists and add the verilog netlist of the design from the dft/do_synth/results directory. Moreover, add the nangate library in verilog format in 0_FreePDK45/Verilog/NangateOpenCellLibrary.v directory.

![Screenshot 2024-12-21 211648](https://github.com/user-attachments/assets/5f17e01c-b172-4c96-a39f-dba922705cd3)

**5.3. Build the design**

The Run Build Model dialog box sets the relevant parameters and builds the in-memory simulation model from the design modules that have been read in. TetraMAX ATPG performs a learning process, which I can control with this dialog. If a simulation model already exists, it is automatically deleted before the new model is built. Select the Build from the menu bar and select top design. I use default options of Set learning box.

![Screenshot 2024-12-21 211829](https://github.com/user-attachments/assets/72c442d3-e23c-4fb8-9512-bed1dd3fd4f6)


**5.4. DRC check**

Now, I perform design rule checking (DRC). First, specifie the name of the STIL procedures file that defines the scan chains and test procedures, then click on Run.

![Screenshot 2024-12-21 211955](https://github.com/user-attachments/assets/bc381f4a-8918-41bf-ae98-ea0dc2217077)

**5.5. Generate the test pattern**

After I have imported the design and completed the DRC check without error, I can use TetraMax to create the test patterns. First, the errors should be annotated. In this case, I would like to check all stuck-at errors. Use the following two commands to annotate the errors and generate the test patterns:

add_faults -all

run_atpg -auto

![Screenshot 2024-12-21 212346](https://github.com/user-attachments/assets/1d34470a-19ad-4f49-a877-0cf71919941f)


Finally, I will write the generated pattern in binary format using the following tcl command:

write_patterns my_adder_pattern.v -internal -format binary



