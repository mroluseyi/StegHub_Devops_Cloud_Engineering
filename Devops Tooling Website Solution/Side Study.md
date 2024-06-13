### SIDE STUDY DOCUMENTATION

### Network Attached Storage (NAS)

Network Attached Storage (NAS) is a dedicated file storage device that provides local area network (LAN) users with centralized, consolidated disk storage through a standard Ethernet connection. NAS systems are popular due to their ease of use, efficiency in sharing storage among multiple users, and their ability to scale out by adding more NAS devices.

- Easy to manage: NAS devices are typically simple to set up and manage, often requiring minimal IT intervention.

- File-level storage: NAS systems operate at the file level, meaning they store and manage data as files.

- Accessibility: Accessible to multiple clients over the network using standard protocols like NFS, SMB, and FTP.

### Storage Area Network (SAN)

A Storage Area Network (SAN) is a high-speed network of storage devices that also connects to servers. Unlike NAS, SAN provides block-level storage that can be accessed by servers as if it were local storage.

- High performance: Designed for high-speed data transfer and low latency, making it ideal for high-demand applications like databases.

- Block-level storage: SAN operates at the block level, enabling high performance for transactional applications.

- Scalability: SANs can be scaled up by adding more storage devices or expanding the network infrastructure.

### Related Protocols

#### Network File System (NFS)

- Description: A protocol that allows file access over a network. Commonly used with UNIX and Linux systems.

- Use case: Ideal for environments where files need to be accessed and shared across multiple clients.

#### Server Message Block (SMB)

- Description: A network file sharing protocol that allows applications to read and write to files and request - services from server programs in a computer network. Used predominantly in Windows environments.

- Use case: Common in Windows networks for file and printer sharing.

#### File Transfer Protocol (FTP) and Secure File Transfer Protocol (SFTP)

- FTP: A standard network protocol used for the transfer of files from one host to another over a TCP-based network.

- SFTP: An extension of the SSH protocol that provides secure file transfer capabilities.

#### Internet Small Computer System Interface (iSCSI)

- Description: An IP-based storage networking standard for linking data storage facilities. iSCSI allows SCSI commands to be sent over IP networks.

- Use case: Commonly used to facilitate data transfers over intranets and to manage storage over long distances.

### Block-level Storage

Block-level storage refers to data storage that manages data as individual blocks. Each block can be controlled as an individual hard drive and is managed by the storage infrastructure.
