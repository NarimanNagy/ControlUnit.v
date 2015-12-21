
`timescale 1ns / 1ps

module ControlUnit
(
	instruction,
	data_mem_wren,
	reg_file_wren,
	reg_file_dmux_select,
	reg_file_rmux_select,
	AluSrc,
	AluOp,
	alu_zero,
	pc_control
);

    input 		[31:0]		instruction;
	 input 					alu_zero; 
		
    output 		[3:0]		data_mem_wren;
	output					reg_file_wren;
	output					reg_file_dmux_select;
	output					reg_file_rmux_select;
	output					AluSrc;
	output		[3:0]		AluOp;
	output		[2:0]		pc_control;
	
   
	reg [3:0]	data_mem_wren;
	reg 		reg_file_wren;
	reg 		reg_file_dmux_select;
	reg 		reg_file_rmux_select;
	reg 		AluSrc;
	reg [3:0] 	AluOp;
	reg [2:0] 	pc_control;
    

	reg [25:0] address;
	reg [5:0] funct;
	reg [15:0] immediate;
    reg [4:0] rs;
    reg [4:0] rt;
    reg [4:0] rd;
    reg [4:0] shamt;
    reg [2:0] type;
	
	wire [5:0] op;
	

	
	assign op = instruction[31:26]; // Set the opcode
	
	always @(instruction)
	begin
	
		if (op == 6'b000000)// R format
	  begin 
			address		= 26'b00000000000000000000000000;
			immediate	= 16'b0000000000000000;
			rs			= instruction[25:21];
			rt			= instruction[20:16];
			rd			= instruction[15:11];
			shamt		= instruction[10:6];
			type		= 3'b001;
			funct		= instruction[5:0];
		end
		else if (op == 6'b000010 || op == 6'b000011) // J format
		begin
			address		= instruction[25:0];
			immediate	= 16'b0000000000000000;
			rs			= 5'b00000;
			rt			= 5'b00000;
			rd			= 5'b00000;
			shamt		= 5'b00000;
			type		= 3'b100;
			funct		= 6'b000000;
		end 
		else  // I format
		begin
			address		= 26'b00000000000000000000000000;
			immediate	= instruction[15:0];
			rs			= instruction[25:21];
			rt			= instruction[20:16];
			rd			= 5'b00000;
			shamt		= instruction[10:6];
			type		= 3'b010;
			funct		= instruction[5:0];
		end
		
//*********************************************************************************************
//data_mem_wren
		
		if (op == 6'h2B) begin // store word
			
			/*if (immediate[2:0] == 0) begin // bytes 0-3
				data_mem_wren = 4'b1111;
			end else if (immediate[2:0] == 1) begin // bytes 0-2
				data_mem_wren = 4'b0111;
			end else if (immediate[2:0] == 2) begin // bytes 0-1
				data_mem_wren = 4'b0011;
			end else if (immediate[2:0] == 3) begin // bytes 0
				data_mem_wren = 4'b0001;
			end*/
			
			data_mem_wren = 4'b1111;
			
		end
		else  // no write
		begin
			data_mem_wren = 4'b0000;
		end
		
		//*****************************************************************************************
		//reg_file_wren
		
		if (op == 6'h4 || op == 6'h5 || op == 6'h2 || op == 6'h3 || (op == 6'h0 && funct == 6'h8)) // branch
		begin 
			reg_file_wren = 0;
		end 
		else // Otherwise 1
		begin 
			reg_file_wren = 1;
		end
		
		//******************************************************************************************************
		//reg_file_dmux_select

		if (op == 6'h23 ||op == 6'h21 ||op == 6'h25 ||op == 6'h20 ||op == 6'h24)// variations of load word
		begin 
			reg_file_dmux_select = 0;
		end
		else 
		begin
			reg_file_dmux_select = 1;
		end
		
		if (op == 6'h0)  // R format
		begin
			reg_file_rmux_select = 1;
		end 
		else // I format or other
		begin 
			reg_file_rmux_select = 0;
		end
		
		
   	//******************************************************************************************************************
		//  AluSrc
		
		if (!(op == 6'h0 || op == 6'b000010 || op == 6'b000011)) begin // I format
			AluSrc = 1;
		end else begin // R or J format
			AluSrc = 0;
		end
		
		
		
		
		
		
		// ***********************************************************************************************
		//  Aluop
		
		if ((op == 6'h0 && funct == 6'h24) || op == 6'hC)  // AND
		begin
			AluOp = 4'b0000;
		end 
		
		else if ((op == 6'h0 && funct == 6'h25) || op == 6'hD)// OR
		begin 
			AluOp = 4'b0001;
		end 
		
		else if ((op == 6'h0 && funct == 6'h21) || op == 6'h9)  // ADD - unsigned
		begin
			AluOp = 4'b0010; 
		end 
		
		else if ((op == 6'h0 && funct == 6'h26)) // XOR
		begin
			AluOp = 4'b0011;
		end
		
		else if ((op == 6'h0 && funct == 6'h27)) // NOR
		begin
			AluOp = 4'b0100;
		end
		
		else if ((op == 6'h0 && funct == 6'h22)) // SUBTRACT - unsigned
		begin
			AluOp = 4'b0110;
		end
		
		else if ((op == 6'h0 && funct == 6'h2A) || op == 6'hA) // SLT
		begin 
			AluOp = 4'b0111;
		end
		
		else if ((op == 6'h0 && funct == 6'h0)) // SLL
		begin
			AluOp = 4'b1000;
		end else if ((op == 6'h0 && funct == 6'h2)) begin // SRL
			AluOp = 4'b1001;
		end else if ((op == 6'h0 && funct == 6'h3)) begin // SRA
			AluOp = 4'b1010;
		end else if ((op == 6'h0 && funct == 6'h20) || op == 6'h8 || op == 6'h2B || op == 6'h23) begin  // ADD - signed
			AluOp = 4'b1011;
		end else if ((op == 6'h0 && funct == 6'h22) || op == 6'h4 || op == 6'h5) begin // SUB - signed - AND bne, beq
			AluOp = 4'b1100;
		end else begin
			AluOp = 4'b1111;
		end		
		
		
		// *********************************************************************************************
		//  pc_control
		
		// NOTE: the delay is to avoid a race condition
		#0.5 if (op == 6'h2 || op == 6'h3) // j, jal
		begin
			pc_control = 3'b001;
		end 
		
		else if (op == 6'h0 && funct == 6'h8)// jr
		begin 
			pc_control = 3'b010;
		end
		
		else if (op == 6'h4 && alu_zero == 1)// beq
		begin 
				$display("###beq");
				pc_control = 3'b011;
		end 

		else if (op == 6'h5 && alu_zero == 0) // bne
		begin
				$display("###bne");
				pc_control = 3'b011;
		end
		
		else  // Default
		begin
			pc_control = 3'b000;
		end
		
	end
 endmodule 
