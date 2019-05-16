/*      Bailey Wellen
        Computer Organization Spring 2019
        Project 6 - File Corruption */

        
#include <iostream>
#include <fstream>
#include <sys/mman.h>
#include <unistd.h>
#include <fcntl.h>
#include <vector>
#include <algorithm>
//#include "my_stat.h"
#include "fs.h"
#include "types.h"
using namespace std;

/* FILES THAT I KNOW THINGS ABOUT 

1- . or .. inode is not a directory 
2- Directory 17 has an incorrect directory link count 
3- File 2 has an incorrect file link count 
4- invalid superblock size
5-
6- Missing Inode - Inode 64 is found in a valid directory but is not marked as allocated in the inode array.
7- inode 80 has an invalid size 
8- Block 65 is referenced by 2 inodes 
9- inode 0 is allocated
10-
11-
12- File 3 has an incorrect file link count 
13- Unused inode - inode 13 is marked as valid but not referred to in a directory
14- non-root directory lists itself as inode 1
15-
16- invalid number of blocks listed 

good- none!!! 

*/
struct myblock
{
    uint block_num;
    //marked_allocated - if the bitmap array says that the data block is allocated, this will be true 
    bool marked_allocated = false;
    uint times_referenced = 0;
};

struct myinode
{
    uint inode_num;
    uint listed_link_count;
    uint actual_link_count;
    uint type;
    //bool marked_allocated - if it is 
    bool marked_allocated;
    //bool in-dir - if it is referenced in a directory, mark this as true 
    bool in_dir = false;
};

struct mydirectory
{
    uint inode_num;
    uint listed_link_count;
    uint actual_link_count;
    uint type;
};

std::ifstream::pos_type filesize(const char* filename)
{
    std::ifstream in(filename, std::ifstream::ate | std::ifstream::binary);
    return in.tellg(); 
}
//pass inode_valid_size the array of inodes 
bool inode_valid_size(dinode* d[], uint size )
{   
    bool all_valid_sizes = true;
    //check if each one's -size is < (512 * MAXFILE)
    for (uint i = 0; i < size; i++)
    {
        if((d[i])->size > (512 * MAXFILE))
        {
            cout << "inode " << i << " has an invalid size " << endl;
            all_valid_sizes = false;
            break;
        }
    }
    return all_valid_sizes;
}
// void check_types(dinode* d[], uint size)
// {
//     for (uint i = 0; i <size; i++)
//     {
//         cout << "type is " <<  d[i]->type << endl;
//     }
// }
bool valid_size(superblock* s, uint size)
{
    bool ret = true;
    //convert from bytes into blocks
    size = size / BSIZE;
    if (s->size != size)
    {
        cout << "superblock contains an invalid size" << endl;
        ret = false;
    }
    return ret;
}
//check values listed in the superblock
bool check_number_of_blocks(superblock* s)
{
    bool ret = true;
    if((s->nblocks + s->bmapstart + 1) != s->size)
    {
        ret = false;
        cout << "There is an invalid number of blocks listed in the superblock. The number should be " << (s->size - s->bmapstart - 1) << endl;
    }
    return ret;
}
//fill all four vectors - one of your directories and one of files, and then special and unused 
void set_vectors(vector <dinode*> &directories,  vector <myinode> &my_inode_unused, vector <dinode*> &special, dinode * d[], vector <myinode> &my_inode_files, vector <mydirectory> &my_directories, uint size)
{
    cout << "SIZE: " << size << endl;
    for(uint i = 0; i < size; i++)
    {
        //Unused
        if (d[i]->type == 0)
        {
            myinode temp;
            temp.inode_num = i;
            temp.listed_link_count = d[i]->nlink;
            temp.type = d[i]->type;
            my_inode_unused.push_back(temp);
            //unused.push_back(d[i]);
        }
        //Directory
        else if(d[i]->type == 1)
        {
            mydirectory temp;
            temp.listed_link_count = d[i]->nlink;
            temp.inode_num = i;
            temp.type = d[i]->type;
            my_directories.push_back(temp);
            directories.push_back(d[i]);
        }
        //File
        else if(d[i]->type == 2)
        {
            myinode temp;
            temp.inode_num = i;
            temp.listed_link_count = d[i]->nlink;
            temp.type = d[i]->type;
            my_inode_files.push_back(temp);
           // files.push_back(d[i]);
        }
        //Special 
        else if(d[i]->type == 3)
        {
            special.push_back(d[i]);
        }

        // for(uint j = 0; j< NDIRECT; j++)
        // {
        //     blocks.push_back(d[i]->addrs[j]);
        // }
    }
}
bool check_inode_0(dinode* d[])
{
    bool ret = true;
    if (d[0]->type != 0)
    {
        cout << "Inode 0 should not be allocated, but is" << endl;
         ret = false;
    }
    return ret;
}
//check if inode one is not a directory 
bool check_inode_1(dinode * d[])
{
    bool ret = true;
    if (d[1]->type != 1)
    {
        cout << "Inode 1 is not a directory " << endl;
         ret = false;
    }
    return ret;
}
//write a function that returns a bool - if the bit is 1, it returns true. if the bit is 0, it returns false 
//bmap is a pointer to the bitmap
//b is the 
//ninodes is the number of inodes 
struct find_bit
{
    uint block;
    uint byte;
    uint bit;
};
find_bit byte_offset(uint b, uint ninodes)
{
    find_bit this_bit;
    //which block in BM?
   // this_bit.block  = BBLOCK(b, ninodes) * BSIZE;
    //which byte in that block? 
    this_bit.byte = b / 8;

    //which bit in that byte?
    //use num % 8 to compute which bit s
    this_bit.bit = b % 8;

    return this_bit;
}

bool bit_is_on(uchar* bitmap_ptr, uint b, uint ninodes )
{
    find_bit offset = byte_offset(b, ninodes);
    uchar* this_bit = bitmap_ptr + offset.block + offset.byte;
    //byte&(0x1 << bn)
    //return b & (0x1 << 1);
    //where bn is how many times you want to shift it over 
    bool bit_on = false;
    if (((*this_bit >> offset.bit)& 0x01))
    {
        bit_on = true;
    }

    return bit_on;
}
// bool check_directories(vector <dinode*> inode_directories)
// {
//     vector <dirent*> directories;
//     for(uint i = 0; i < inode_directories.size(); i++)
//     {
//         dirent* temp = inode_directories.at(i)->addrs;   
//     }
// }
//make sure that the first two file names are . and .. 
bool check_names(vector <dirent*> dirent_vec)
{
    bool ret = true;
    //check if the first name is "."
    if(dirent_vec.at(0)->name[0] != '.')
    {
        cout << "Directory " << dirent_vec.at(0)->inum << " does not have a first directory of \".\" " << endl;
        ret = false;
    }
    //check if the second name is ".."
    else if (dirent_vec.at(1)->name[0] != '.' || dirent_vec.at(1)->name[1] != '.')
    {
        cout << "Directory " << dirent_vec.at(1)->inum << " does not have a second directory of \"..\" " << endl;
    }
    return ret;
}
bool check_nonroot(vector <dirent*> dirent_vec)
{
    bool ret = true;
    //skip the beginning "." and ".." files 
    for(uint i = 2; i < dirent_vec.size(); i++)
    {
        //check if name is . and inode number is 1 
         if(dirent_vec.at(i)->name[0] == '.' && dirent_vec.at(i)->name[1] != '.' && dirent_vec.at(i)->inum == 1)
        {
            cout << "A non-root directory lists \".\" (itself) as Inode 1 " << endl;
            ret = false;
            break;
        }
    }
    return ret;
}
//directory appearing more than once not including . and ..
bool check_repeat_directory(vector <dirent*> dirent_vec)
{
    vector <char*> names;
    bool ret = true;
    for(uint i = 0; i < dirent_vec.size(); i++)
    {
        //if the name is "." or "..", don't worry about it 
        if(dirent_vec.at(i)->name[0] == '.')
            continue;
        //if there is a name that is repeated, return false 
        if (std::find(names.begin(), names.end(), dirent_vec.at(i)->name) != names.end())
        {
            cout << "Directory " << dirent_vec.at(i)->name << " appears more than once. " << endl;
            ret = false;
            break;
        }
        else
        {
            names.push_back(dirent_vec.at(i)->name);
        }
        
    }
    return ret; 
}
bool check_multiply_allocated_block(dinode* d[], uint size)
{
    //vector <myaddrs> addresses;
    vector <uint> addresses;
    bool ret = true; 
    for(uint i = 0; i < size; i++)
    {
        //cout << endl << "address : " ;
        for(uint j = 0; j < NDIRECT; j++)
        {
            if (std::find(addresses.begin(), addresses.end(), d[i]->addrs[j]) != addresses.end() && d[i]->addrs[j] != 0)
            {
                cout << "Multiply Allocated Block : Block " <<  d[i]->addrs[j] << " is referenced by inode " << i << " and another inode " << endl;
                ret = false;
                goto leave; 
            }
            else
            {
                addresses.push_back(d[i]->addrs[j]);
            }
        }
    }

leave:
    return ret;
}
bool check_if_should_be_directory(vector<dirent*> dirent_vec, dinode* d[])
{
    bool ret = true;
    for (uint i = 0; i < dirent_vec.size(); i++)
    {
        // if name is . or .. 
        if(dirent_vec.at(i)->name[0] == '.')
        {
            if(d[dirent_vec.at(i)->inum]->type != 1)
            {
               cout << "Name of File in inode " << dirent_vec.at(i)->inum << " is \".\" or \"..\" but does not lead to a directory" << endl;
               ret = false;
               break;
            }
        }
    }
    return ret;
}

//check that the number of times a file is referenced is equal to the listed link count 
bool check_file_link_count(vector <dirent*> dirent_vec, vector <myinode> my_inode_files)
{
    bool ret = true;
    for(uint i = 0; i < my_inode_files.size(); i++)
    {
        uint listed_count  = my_inode_files.at(i).listed_link_count;
        //cout << "Inode Link Count is " << my_inode_files.at(i).listed_link_count << endl;
        uint actual_count = 0;
        for(uint j = 0; j <dirent_vec.size(); j++)
        {
            //count the number of times that directories reference the file 
            if(dirent_vec.at(j)->inum == my_inode_files.at(i).inode_num)
               actual_count++;
        }

        if(actual_count != listed_count)
        {
                cout << "File " << my_inode_files.at(i).inode_num << " has an incorrect file link count. The correct number is " << actual_count << endl;
                ret = false;
                break;
        }        
    }
    return ret;
}
//check that the number of times a directory is referenced is the same as it is listed as 
bool check_directory_link_count(vector <dirent*> dirent_vec, vector <mydirectory> my_directories)
{
    bool ret = true;
    for(uint i = 0; i < my_directories.size(); i++)
    {
        uint listed_count = my_directories.at(i).listed_link_count;
        uint actual_count = 0;
        for(uint j = 0; j <dirent_vec.size(); j++)
        {
            //count the number of times that directories reference the directory 
            if(dirent_vec.at(j)->inum == my_directories.at(i).inode_num)
               actual_count++;
        }

         if(actual_count != listed_count)
        {
                cout << "Directory " << my_directories.at(i).inode_num << " has an incorrect directory link count. The correct number is " << actual_count << endl;
                ret = false;
                break;
        }     
    }
    return ret;
}
//An inode marked valid is not referred to by a directory.
bool check_unused_inode(vector <myinode> inode_vec)
{
    bool ret = true;
    for(uint i = 0; i < inode_vec.size(); i++)
    {
        if(inode_vec.at(i).type != 0 && inode_vec.at(i).in_dir == false)
        {
            cout << "Unused Inode: Inode Number " << inode_vec.at(i).inode_num << " is marked as valid but is not referred to by a directory. " << endl;
            ret = false;
            break;
        }
    }
    return ret;
}
//An inode number is found in a valid directory but is not marked as allocated in the inode array.
bool check_missing_inode(vector <myinode> unused_inode_vec)
{
    bool ret = true;
    for(uint i = 0; i < unused_inode_vec.size(); i++)
    {
        if(unused_inode_vec.at(i).type == 0 && unused_inode_vec.at(i).in_dir == true)
        {
            cout << "Missing Inode: Inode Number " << unused_inode_vec.at(i).inode_num << " is found in a valid directory but is not marked as allocated in the inode array." << endl;
            ret = false;
            break;
        }
    }
    return ret;
}

//The bit map says block n is allocated but you cannot find it belonging to any file or directory.
bool check_missing_block(myblock bits[], uint size)
{
    bool ret = true;
    for(uint i = 0; i < size; i++)
    {
        //something is not working here 
        if (bits[i].marked_allocated == true && bits[i].times_referenced == 0)
        {
            cout << "Missing Block: Block number " << bits[i].block_num << " is marked allocated in the bitmap but is not referred to by a file or directory." << endl;
            ret = false;
            break;
        }
    }
    return ret;
}
//The bit map says block n is free but you found it referenced by a valid inode.
bool check_unallocated_block(myblock bits[], uint size)
{
    bool ret = true;
    for(uint i = 0; i < size; i++)
    {
        if (bits[i].marked_allocated == false && bits[i].times_referenced > 0)
        {
            cout << "Unallocated Block: Block number " << bits[i].block_num << " is marked as free in the bitmap but is referred to by an inode " << endl;
            ret = false;
            break;
        }
    }
    return ret;
}

int main(int argc, char* argv[])
{
	if (argc < 2)
	{
		cout << "not enough arguments entered" << endl;
		exit(1);
	}
    char* file = argv[1];
    int fd = open(file, O_RDONLY);
	if (fd == -1)
	{
		cout << "invalid file" << endl;
		exit(1);
	}

    void * file_copy = mmap(0, BSIZE * 1024, PROT_READ, MAP_SHARED, fd, 0);
    superblock * sb = (superblock*)(((char*)file_copy) + BSIZE);
    cout << "File System Project " << endl;
    cout << "Super Block Information : " << endl;
    cout << "Size of File System: " << sb->size << endl;
    cout << "Number of Inodes: " <<  sb->ninodes << endl;
    cout << "Number of Blocks: " << sb->nblocks << endl;
    cout << "Number of Log Blocks: " << sb->nlog << endl;
    cout << "Block of log start: " << sb->logstart << endl;
    cout << "Block of Inode Start: " << sb->inodestart << endl;
    cout << "Block of Bitmap Start: " << sb->bmapstart << endl;

    cout << endl;


    //dinode * i_ptr = (dinode*)(((char*)sb) + (sb->nlog * BSIZE) + BSIZE);
    dinode* i_ptr = (dinode*)((char*)file_copy + (sb->inodestart * BSIZE));
    dinode* dinode_array[sb->ninodes];
     for(uint i = 0; i < sb->ninodes - 1; i++)
     {
         dinode_array[i] = i_ptr;
        // cout << "inode " << i << " has link count "  << dinode_array[i]->nlink << " and size of " << dinode_array[i]->size << " and type " << dinode_array[i]->type << endl;
         i_ptr += 1;
     }

     //first two checks 
     //CHECK to make sure we have a valid sized file
     //if we don't check this here, the loops later may be invalid if the sizes are not good
    int size = filesize(argv[1]); 
   if (valid_size(sb, size) == false)
        exit(1);
    //CHECK that all inode sizes are valid 
    if(inode_valid_size(dinode_array, sb->ninodes - 1) == false)
        exit(1);

    
    uchar* bitmap_ptr = (uchar*)((char*)file_copy + (sb->bmapstart * BSIZE));
    
    uchar* bitmap[sb->size / 8];

    //myinode bmap_inodes[BSIZE * 8];
    myblock bitmap_blocks[sb->size];

    //bool bmap[BSIZE * 8];

    //set bitmap array
    for(uint i = 0; i < sb->size / 8; i++)
    {
        bitmap[i] = bitmap_ptr;
        for(uint j = 0; j < 8; j++)
        {
            bitmap_blocks[(i * 8) + j].marked_allocated = ((*bitmap[i]>>(7 - j))& 0x01);       

            bitmap_blocks[(i * 8) + j].block_num = (i * 8) + j;
        }
       bitmap_ptr++;
    }
    //int counter = 0;

    //set more values of bitmap_blocks vector 
    //cout << "Block numbers referenced : " << endl;
    for(uint i = 0; i < sb->ninodes - 1; i++)
    {
        for(uint j = 0; j < NDIRECT; j++)
        {
            //still need to add indirects to this...
            uint block_n = dinode_array[i]->addrs[j];
            bitmap_blocks[block_n].times_referenced++;  
        }
    }

     //print bit map for viewing purposes
    // for(uint i = 0; i < sb->size; i++)
    // {
    //     cout << bitmap_blocks[i].marked_allocated << "  ";
    //     // cout << bitmap_blocks[i].marked_allocated;
    //     counter++;
    //     if(i  % 8 == 0)
    //     {
    //         cout << "\t" <<  i  << endl;

    //     }
    // }

   //vector <dinode*> inode_files;
   vector <myinode> my_inode_files;
    vector <mydirectory> my_directories;

   vector <dinode*> inode_directories;
//    vector <dinode*> inode_unused;
   vector <myinode> inode_unused;

   vector <dinode*> inode_special;

   // uint check_used[sb->ninodes] = {0};    

   set_vectors(inode_directories,  inode_unused, inode_special, dinode_array, my_inode_files, my_directories, sb->ninodes);
   cout << "directories: " <<  inode_directories.size() << endl;
   //cout << "files: " <<  inode_files.size() << endl;

   //LOAD DIRECTORIES
   vector <dirent*> dirent_vec;

    //loop through the inodes that are directories 
   for (uint i = 0; i < inode_directories.size(); i++)
   {
       //Every directory size has to be a multiple of 512 - check that here 
       for(uint k = 0; k < 11; k++)
       {
            dirent* data_section = (dirent*) (((uchar*) file_copy) + inode_directories[i]->addrs[k] * BSIZE);
            //the data_end pointer points to the end of that block 
            for(dirent* ptr = data_section; ptr != data_section + (BSIZE / sizeof(dirent)); ptr++)
            {
                //as long as it is not used, we should push each directory into the vector 
                if(ptr->inum != 0)
                {
                    dirent_vec.push_back(ptr);
                    //cout << "the Inum is " << ptr->inum << " the name is " << (string)ptr->name << endl;
                }        
            }
       }
   }
   //go through and check off any inode files that are referenced in directories 
   for(uint i = 0; i < my_inode_files.size(); i++)
   {
       for(uint j = 0; j < dirent_vec.size(); j++)
        {
            if(dirent_vec.at(j)->inum == my_inode_files.at(i).inode_num)
            {
                my_inode_files.at(i).in_dir = true; 
            }
        }
   }
   //go through and check off any unused inode that are referenced in directories 
   for(uint i = 0; i < inode_unused.size(); i++)
   {
       for(uint j = 0; j < dirent_vec.size(); j++)
        {
            if(dirent_vec.at(j)->inum == inode_unused.at(i).inode_num)
            {
                inode_unused.at(i).in_dir = true; 
            }
        }
   }
    
   //remaining  checks 

    //check superblock first
     //then check root directory

    if(check_unused_inode(my_inode_files) == false)
        exit(1);
    if(check_missing_inode(inode_unused) == false)
        exit(1);
    if(check_number_of_blocks(sb) == false)
        exit(1);
    if(check_nonroot(dirent_vec) == false)
        exit(1);
    //CHECK to make sure that inode 1 is a directory 
    if(check_inode_1(dinode_array) == false )
        exit(1);
    //CHECK to make sure that inode 0 has a type 0 - or that inode 0 is not allocated 
    if(check_inode_0(dinode_array) == false)
        exit(1);
   //CHECK that first two file names are . and .. 
    if(check_names(dirent_vec) == false)
        exit(1);
    if(check_repeat_directory(dirent_vec) == false)
        exit(1);
    //check if a block is referenced by more than one inode 
    if(check_multiply_allocated_block(dinode_array, sb->ninodes - 1) == false)
        exit(1);
    if(check_if_should_be_directory(dirent_vec, dinode_array) == false)
        exit(1);
    if (check_file_link_count(dirent_vec, my_inode_files) == false)
        exit(1);
    if(check_directory_link_count(dirent_vec, my_directories) == false)
        exit(1);
    

    //the ones below do not seem to be working - probably need to consider indirects 
    // if(check_missing_block(bitmap_blocks, sb->size) == false)
    //      exit(1);
    // if(check_unallocated_block(bitmap_blocks, sb->size) == false)
    //     exit(1);
    

   



    close(fd);
    munmap(file_copy, BSIZE * 1024);
	return 0;
}
