`timescale 1ns / 1ps


module CACHE_CONTROLLER(address,clk,data,mode,output_data,hit1, hit2);


parameter no_of_address_bits=11;  
parameter no_of_blkoffset_bits=2;
parameter byte_size=8;          //one block is of 8 bits //

parameter no_of_l2_ways=4;      //4-way set-associative
parameter no_of_l2_ways_bits=2;     //2 bits are sufficient to represent 4 values
parameter no_of_l2_blocks=64;       //No. of blocks in L2 Cache
parameter no_of_bytes_l2_block=4;       //No. of bytes in a L2 Cache block= 4
parameter l2_block_bit_size=32;          // 8*4=32
parameter no_of_l2_index_bits=4;        // 2^4=16
parameter no_of_l2_tag_bits=5;          //No. of tag bits= Address_bits - index_bits- Block_offset = 11 -4 -2 =5
parameter no_of_l2_set=16;

parameter no_of_l1_blocks=8;        // No. of lines in L1 Cache... as one line contains 1 block...it is equal to no. of blocks
parameter no_of_bytes_l1_block=4;       //Each Block has 4 bytes
parameter l1_block_bit_size=32;         //Size of each line = No. of blocks in a line * No. of bytes in a block * Byte_size = 1*4*4=16
parameter no_of_l1_index_bits=3;        //as 2^3=8... So 3 index bits are sufficient to locate a line on L1 Cache
parameter no_of_l1_tag_bits=6;          //No. of tag bits= Address_bits - index_bits- Block_offset = 11 -3 -2 =6

parameter no_of_main_memory_blocks=512; //2^5 //No. of lines in main_memory... as each line contains a single block... No. of lines=No. of blocks here
parameter main_memory_block_size=32;    //BIT SIZE//Each line has one block... which in turn has 4 bytes and each byte is of 4 bits=1*4*4=16
parameter no_of_bytes_main_memory_block=4;   //Each line has one block and each block has 4 bytes
parameter main_memory_byte_size=2048;        //No. of bytes in main memory=No. of lines* No. of bytes in each line=32*4=128


input [no_of_address_bits-1:0]address;
input clk;
input [byte_size-1:0]data;
input mode;                 //mode=0 : Read     mode=1 : Write
output reg[byte_size-1:0]output_data;
output reg hit1, hit2;              


reg [no_of_address_bits-1:0]address_valid;          //For Checking whether there is a stored block at some line in Cache or not
reg [no_of_address_bits-no_of_blkoffset_bits-1:0]main_memory_blk_id;        //Represents the line number to which the address belongs on main memory
reg [no_of_l1_tag_bits-1:0]l1_tag;          //The tag for lines on L1 Cache
reg [no_of_l1_index_bits-1:0]l1_index;      //Represents the index of the line to which the address belongs on L1 Cache
reg [no_of_l2_tag_bits-1:0]l2_tag;      //Represents the index of the line to which the address belongs on L1 Cache
reg [no_of_l2_index_bits-1:0]l2_index;          //The index of the line to which the address belongs on L2 Cache
reg [no_of_blkoffset_bits-1:0]offset;           //Offset gives the index of byte within a block

//
integer i;                  //integer variables for working in for-loops
integer j;
//
//the variable given below in various search operation in L1 , L2 and main memory
//specially when we need to evict some block from L1 or L2 Cache
//then it needs to be searched in the L2 or in main memory to update its value there
integer l2_check;
integer l2_check2;
integer l2_checka;
integer l2_check2a;
integer l2_mm_check;
integer l2_mm_check2;
integer l2_mm_iterator;
integer l2_iterator;

integer l1_l2_check;
integer l1_l2_check2;
integer l1_l2_checka;
integer l1_l2_check2a;
integer l1_l2_checkb;
integer l1_l2_check2b;
//
//Many times we need to evict an block from L1 or L2 Cache..
//so its value needs to be updated in L2 or main Memory
//these are the variable used for evicting operations
//for finding the block present in L1 or L2..its location in L2 or main memory
reg [no_of_l2_ways_bits-1:0]lru_value;
reg [no_of_l2_ways_bits-1:0]lru_value_dummy;

reg [no_of_l2_ways_bits-1:0]lru_value2;
reg [no_of_l2_ways_bits-1:0]lru_value_dummy2;

reg [no_of_l1_tag_bits-1:0]l1_evict_tag;
reg [no_of_l2_tag_bits-1:0]l1_to_l2_tag;
reg [no_of_l2_index_bits-1:0]l1_to_l2_index;

reg [no_of_l1_tag_bits-1:0]l1_evict_tag2;
reg [no_of_l2_tag_bits-1:0]l1_to_l2_tag2;
reg [no_of_l2_index_bits-1:0]l1_to_l2_index2;

reg [no_of_l1_tag_bits-1:0]l1_evict_tag3;
reg [no_of_l2_tag_bits-1:0]l1_to_l2_tag3;
reg [no_of_l2_index_bits-1:0]l1_to_l2_index3;

reg [no_of_l2_tag_bits-1:0]l2_evict_tag;
//
//to store whether the block to be evicted was found in L2 or main memory or not
reg l1_to_l2_search;
reg l1_to_l2_search2;
reg l1_to_l2_search3;

reg dummy_hit;

reg dummy_hit_w=0;
reg [no_of_address_bits-1:0] stored_address;           
reg stored_mode;                
reg [byte_size-1:0]stored_data;        
//MAIN_MEMORY main_memory_instance();
reg [main_memory_block_size-1:0]main_memory[0:no_of_main_memory_blocks-1];
initial 
begin: initialization_main_memory
    integer i;
    for (i=0;i<no_of_main_memory_blocks;i=i+1)
    begin
        main_memory[i]=i;
    end
end

//L1_CACHE_MEMORY l1_cache_memory_instance();
reg [l1_block_bit_size-1:0]l1_cache_memory[0:no_of_l1_blocks-1];
reg [no_of_l1_tag_bits-1:0]l1_tag_array[0:no_of_l1_blocks-1];
reg l1_valid[0:no_of_l1_blocks-1];

initial 
begin: initialization_l1
    integer i;
    for  (i=0;i<no_of_l1_blocks;i=i+1)
    begin
        l1_valid[i]=1'b0;
        l1_tag_array[i]=0;
    end
end

//L2_CACHE_MEMORY l2_cache_memory_instance();
reg [(no_of_l2_ways)*l2_block_bit_size-1:0]l2_cache_memory[0:no_of_l2_set-1];
reg [no_of_l2_tag_bits*no_of_l2_ways-1:0]l2_tag_array[0:no_of_l2_set-1];//changed
reg [no_of_l2_ways-1:0]l2_valid[0:no_of_l2_set-1];//change samel2_valid[0:16]
reg [no_of_l2_ways*no_of_l2_ways_bits-1:0]lru[0:no_of_l2_set-1];//change samelru[0:16(no.of sets)]

initial 
begin: initialization
    integer i;
    for  (i=0;i<no_of_l2_blocks;i=i+1)
    begin
        l2_valid[i]=0;
        l2_tag_array[i]=0;
        lru[i]=8'b11100100;
    end
end

always @(posedge clk)
begin

       
            $display("start");
            stored_address=address;
            
            stored_mode=mode;
            stored_data=data;
      
    main_memory_blk_id=(stored_address>>no_of_blkoffset_bits)%no_of_main_memory_blocks;
    l2_index=(main_memory_blk_id)%(no_of_l2_blocks/no_of_l2_ways);
    l2_tag=main_memory_blk_id>>no_of_l2_index_bits;
    l1_index=(main_memory_blk_id)%no_of_l1_blocks;
    l1_tag=main_memory_blk_id>>no_of_l1_index_bits;
    offset=stored_address%no_of_bytes_main_memory_block;
    if (stored_mode==0)
    begin
        $display("rCheck Startedr");
        //
        if (l1_valid[l1_index]&&l1_tag_array[l1_index]==l1_tag)
        begin
            $display("rFound in L1 Cache, L1 hitr");
            output_data=l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size];
            hit1=1;
            hit2=0;
         
            $display("data read successful");
        end
        else
        begin
            //
            $display("rNot Found in L1 Cache, start search in L2 cacher");
            hit1=0;          
            begin //c not found in l1
                hit1=0;
                hit2=1;
               
                dummy_hit=0;
                for (l2_check=0;l2_check<no_of_l2_ways;l2_check=l2_check+1)
                begin
                    $display("flag");
                    if (l2_valid[l2_index][l2_check]&&l2_tag_array[l2_index][((l2_check+1)*no_of_l2_tag_bits-1)-:no_of_l2_tag_bits]==l2_tag)
                    // lru array is 16*8 
                    // l2 tag array is 16*20
                    begin
                        dummy_hit=1;
                        l2_check2=l2_check;
                    end
                end
                if (dummy_hit==1) $display("rL2 hit, Found in L2 Cache in way %D",l2_check2);
                else $display("rNot Found in L2 Cacher");
                if (dummy_hit==1)
                // value of l2_check is from 0 to 3
                begin
                    lru_value2=lru[l2_index][((l2_check2+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits];
                    for (l2_iterator=0;l2_iterator<no_of_l2_ways;l2_iterator=l2_iterator+1)
                    begin
                       lru_value_dummy2=lru[l2_index][((l2_iterator+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits];
                       $display("rLRU found %B",lru[l2_index]);
                       if (lru_value_dummy2>lru_value2)
                       begin
                           lru[l2_index][((l2_iterator+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits]=lru_value_dummy2-1;
                       end
                    end
                    $display("rAfter LRU reduction %B",lru[l2_index]);
                    lru[l2_index][((l2_check2+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits]=no_of_l2_ways-1;
                    $display("rLRU updated to %B",lru[l2_index]);
                    
                    if (l1_valid[l1_index]==0)
                    begin
                        $display("rPlaced in L1 without evictionr");
                        l1_cache_memory[l1_index]=l2_cache_memory[l2_index][((l2_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size];
                        l1_valid[l1_index]=1;
                        l1_tag_array[l1_index]=l1_tag;
                        output_data=l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size];
                        dummy_hit=1;
                        $display("data read successful");
                    end
                    else
                    begin
                    // there is already a data in l1 need to evict and place data of l2 
                        l1_evict_tag2=l1_tag_array[l1_index];
                        l1_to_l2_tag2=l1_evict_tag2>>(no_of_l1_tag_bits-no_of_l2_tag_bits);
                        l1_to_l2_index2={l1_evict_tag2[no_of_l1_tag_bits-no_of_l2_tag_bits-1:0],l1_index};
                        l1_to_l2_search2=0;
                        for (l1_l2_checka=0;l1_l2_checka<no_of_l2_ways;l1_l2_checka=l1_l2_checka+1)
                        begin
                            if (l2_valid[l1_to_l2_index2][l1_l2_checka]&&l2_tag_array[l1_to_l2_index2][((l1_l2_checka+1)*no_of_l2_tag_bits-1)-:no_of_l2_tag_bits]==l1_to_l2_tag2)
                            begin
                                l1_to_l2_search2=1;
                                l1_l2_check2a=l1_l2_checka; //l1_l2_checka holds the value of the way/block which had tag matching
                            end
                        end
                        if (l1_to_l2_search2==1)
                        begin
                            $display("rfound l1 eviction in L2,updated L1 with reqd datar");
                            l2_cache_memory[l1_to_l2_index2][((l1_l2_check2a+1)*l1_block_bit_size-1)-:l1_block_bit_size]=l1_cache_memory[l1_index];
                            l1_cache_memory[l1_index]=l2_cache_memory[l2_index][((l2_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size];
                            l1_valid[l1_index]=1;
                            l1_tag_array[l1_index]=l1_tag;
                            output_data=l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size];
                            dummy_hit=1;
                            $display("data read successful");
                        end
                        else
                        begin
                            $display("rL1 eviction not found in L2, access main memory and update L1 with reqd datar");
                            main_memory[{l1_evict_tag2,l1_index}]=l1_cache_memory[l1_index];
                            l1_cache_memory[l1_index]=l2_cache_memory[l2_index][((l2_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size];
                            l1_valid[l1_index]=1;
                            l1_tag_array[l1_index]=l1_tag;
                            output_data=l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size];
                            dummy_hit=1;
                            $display("data read successful");
                        end
                    end
                end
                //
                else
                begin //not found in l2 going to main memory
                    hit1=0; 
                    hit2=0;
                  
                    //
                    $display("L2 miss, Extracting from main memory");
                    
                    begin //d
                        $display("rCheck for l2 updation from main memoryr");

                        hit1=0;
                        hit2=0;
                     
                        for (l2_mm_check=0;l2_mm_check<no_of_l2_ways;l2_mm_check=l2_mm_check+1)
                        begin
                            if (lru[l2_index][((l2_mm_check+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits]==0)
                            begin
                                l2_mm_check2=l2_mm_check;
                            end
                        end
                        $display("l2 way from main memory%D",l2_mm_check2);
                        lru_value=lru[l2_index][((l2_mm_check2+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits];
                        for (l2_mm_iterator=0;l2_mm_iterator<no_of_l2_ways;l2_mm_iterator=l2_mm_iterator+1)
                        begin
                            lru_value_dummy=lru[l2_index][((l2_mm_iterator+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits];
                            $display("rlru value found %B",lru[l2_index]);
                           if ((lru[l2_index][((l2_mm_iterator+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits])>lru_value)
                           begin
                               lru[l2_index][((l2_mm_iterator+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits]=lru_value_dummy-1;
                               lru_value_dummy=lru[l2_index][((l2_mm_iterator+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits];
                           end
                        end
                        $display("rlru value found after reduction %B",lru[l2_index]);
                        lru[l2_index][((l2_mm_check2+1)*no_of_l2_ways_bits-1)-:no_of_l2_ways_bits]=(no_of_l2_ways-1);
                        $display("rlru value updated to %B",lru[l2_index]);
                        
                        if (l2_valid[l2_index][l2_mm_check2]==0)// valid bit is zero no worries
                        begin
                            $display("rUpdate l2 without evictionr");
                            l2_cache_memory[l2_index][((l2_mm_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size]=main_memory[main_memory_blk_id];
                            l2_valid[l2_index][l2_mm_check2]=1;
                            l2_tag_array[l2_index][((l2_mm_check2+1)*no_of_l2_tag_bits-1)-:no_of_l2_tag_bits]=l2_tag;
                            if (l1_valid[l1_index]==0) // valid bit is zero no worries 
                            begin // data coming from main memo to l2 to l1
                                $display("rUpdate l1 without evictionr");
                                l1_cache_memory[l1_index]=main_memory[main_memory_blk_id];
                                l1_valid[l1_index]=1;
                                l1_tag_array[l1_index]=l1_tag;
                                output_data=l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size];
                                dummy_hit=0; 
                                $display("data read successful");
                            end
                            else
                            begin //evicting from l1
                                l1_evict_tag=l1_tag_array[l1_index];
                                l1_to_l2_tag=l1_evict_tag>>(no_of_l1_tag_bits-no_of_l2_tag_bits);
                                l1_to_l2_index={l1_evict_tag[no_of_l1_tag_bits-no_of_l2_tag_bits-1:0],l1_index};
                                
                                l1_to_l2_search=0;
                                for (l1_l2_check=0;l1_l2_check<no_of_l2_ways;l1_l2_check=l1_l2_check+1)
                                begin
                                    if (l2_valid[l1_to_l2_index][l1_l2_check]&&l2_tag_array[l1_to_l2_index][((l1_l2_check+1)*no_of_l2_tag_bits-1)-:no_of_l2_tag_bits]==l1_to_l2_tag)
                                    begin
                                        l1_to_l2_search=1;
                                        l1_l2_check2=l1_l2_check;
                                    end
                                end
                                if (l1_to_l2_search==1) // if we have the tag matcjing of evicted block then fine 
                                begin
                                    $display("rfound l1 eviction in l2r");
                                    l2_cache_memory[l1_to_l2_index][((l1_l2_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size]=l1_cache_memory[l1_index];
                                    l1_cache_memory[l1_index]=l2_cache_memory[l2_index][((l2_mm_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size];
                                    l1_valid[l1_index]=1;
                                    l1_tag_array[l1_index]=l1_tag;
                                    output_data=l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size];
                                    dummy_hit=0;
                                    $display("data read successful");
                                end
                                else
                                begin
                                    $display("rL1 eviction not found in l2, access main memory");
                                    main_memory[{l1_evict_tag,l1_index}]=l1_cache_memory[l1_index];
                                    l1_cache_memory[l1_index]=l2_cache_memory[l2_index][((l2_mm_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size];
                                    l1_valid[l1_index]=1;
                                    l1_tag_array[l1_index]=l1_tag;
                                    output_data=l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size];
                                    dummy_hit=0;
                                    $display("data read successful");
                                end
                            end
                        end
                        
                        else
                        begin
                            
                            $display("rInitially valid data present in l2, have to evictr");
                            l2_evict_tag=l2_tag_array[l2_index][((l2_mm_check2+1)*no_of_l2_tag_bits-1)-:no_of_l2_tag_bits];
                            main_memory[{l2_evict_tag,l2_index}]=l2_cache_memory[l2_index][((l2_mm_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size];
                            
                            l2_cache_memory[l2_index][((l2_mm_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size]=main_memory[main_memory_blk_id];
                            l2_valid[l2_index][l2_mm_check2]=1;
                            l2_tag_array[l2_index][((l2_mm_check2+1)*no_of_l2_tag_bits-1)-:no_of_l2_tag_bits]=l2_tag;
                            
                            
                            if (l1_valid[l1_index]==0)
                            begin
                                $display("rL1 is empty, update without evictionr");
                                l1_cache_memory[l1_index]=l2_cache_memory[l2_index][((l2_mm_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size];
                                l1_valid[l1_index]=1;
                                l1_tag_array[l1_index]=l1_tag;
                                output_data=l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size];
                                dummy_hit=0;
                                $display("data read successful");
                            end
                            else
                            begin
                                $display("rL1 is not empty, have to evictr");
                                l1_evict_tag3=l1_tag_array[l1_index];
                                l1_to_l2_tag3=l1_evict_tag3>>(no_of_l1_tag_bits-no_of_l2_tag_bits);
                                l1_to_l2_index3={l1_evict_tag3[no_of_l1_tag_bits-no_of_l2_tag_bits-1:0],l1_index};
                                l1_to_l2_search3=0;
                                for (l1_l2_checkb=0;l1_l2_checkb<no_of_l2_ways;l1_l2_checkb=l1_l2_checkb+1)
                                begin
                                    if (l2_valid[l1_to_l2_index3][l1_l2_checkb]&&l2_tag_array[l1_to_l2_index3][((l1_l2_checkb+1)*no_of_l2_tag_bits-1)-:no_of_l2_tag_bits]==l1_to_l2_tag3)
                                    begin
                                        l1_to_l2_search3=1;
                                        l1_l2_check2b=l1_l2_checkb;
                                    end
                                end
                                if (l1_to_l2_search3==1)
                                begin
                                    $display("rfound l1 eviction in l2r");
                                    l2_cache_memory[l1_to_l2_index3][((l1_l2_check2b+1)*l1_block_bit_size-1)-:l1_block_bit_size]=l1_cache_memory[l1_index];
                                    l1_cache_memory[l1_index]=l2_cache_memory[l2_index][((l2_mm_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size];
                                    l1_valid[l1_index]=1;
                                    l1_tag_array[l1_index]=l1_tag;
                                    output_data=l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size];
                                    dummy_hit=0;
                                    $display("data read successful");
                                end
                                else
                                begin
                                    $display("rl1 eviction not found in l2, update in main memoryr");
                                    main_memory[{l1_evict_tag3,l1_index}]=l1_cache_memory[l1_index];
                                    l1_cache_memory[l1_index]=l2_cache_memory[l2_index][((l2_mm_check2+1)*l1_block_bit_size-1)-:l1_block_bit_size];
                                    l1_valid[l1_index]=1;
                                    l1_tag_array[l1_index]=l1_tag;
                                    output_data=l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size];
                                    dummy_hit=0;
                                    $display("data read successful");
                                end
                            end
                        end
                    end    
                end //
            end //    
        end
    end
    else
    // doing write
    begin
        output_data=0;
        $display("wsearching in L1 cachew");
        $display("initial value of L1 tag %b",l1_tag_array[l1_index]);
        $display("address ka l1 tag %b",l1_tag);
        if (l1_valid[l1_index]==1 && l1_tag_array[l1_index]==l1_tag)
        
        begin
            $display("wFound in L1 Cachew");
            l1_cache_memory[l1_index][((offset+1)*byte_size-1)-:byte_size]=stored_data;
          
            hit1=1;
            hit2=0;
        end
        else
        begin  //else not found in L1 starts here/
            //begin searching in L2 and main memory starts here */
                $display("wnot found in L1, searching in L2 cache");
           
                dummy_hit_w=0;
                hit1=0;
                hit2=0;
                for (l2_checka=0;l2_checka<no_of_l2_ways;l2_checka=l2_checka+1)
                begin
                    if (l2_valid[l2_index][l2_checka]&&l2_tag_array[l2_index][((l2_checka+1)*no_of_l2_tag_bits-1)-:no_of_l2_tag_bits]==l2_tag)
                    begin
                        dummy_hit_w=1;
                        hit2=1;
                        hit1=0;
                     
                        l2_cache_memory[l2_index][(l2_checka*l1_block_bit_size+(offset+1)*byte_size-1)-:byte_size]=stored_data;
                    end
                end
                if (dummy_hit_w==0) 
                begin
                    hit1=0;
                    hit2=0;
                    begin
                        $display("wnot found in L2, access main memoryw");
                        hit1=0;
                        hit2=0;
               // storing the value directly to the main memory
                        main_memory[main_memory_blk_id][((offset+1)*byte_size-1)-:byte_size]=stored_data;
                    end
                end
            end /*searching in L2 and Main ends here */
        end    //else not found in L1 ends here/
    end
//end
endmodule
