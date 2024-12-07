---
title: "Collect baseline: Performance best practices & guidelines"
description: Provides steps to collect a performance baseline as guidelines to optimize the performance of your SQL Server on Azure Virtual Machine (VM).
author: bluefooted
ms.author: pamela
ms.reviewer: mathoma
ms.date: 03/25/2021
ms.service: azure-vm-sql-server
ms.subservice: performance
ms.topic: conceptual
tags: azure-service-management
---
# Collect baseline: Performance best practices for SQL Server on Azure VM
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

This article provides information to collect a performance baseline as a series of best practices and guidelines to optimize performance for your SQL Server on Azure Virtual Machines (VMs).

There is typically a trade-off between optimizing for costs and optimizing for performance. This performance best practices series is focused on getting the *best* performance for SQL Server on Azure Virtual Machines. If your workload is less demanding, you might not require every recommended optimization. Consider your performance needs, costs, and workload patterns as you evaluate these recommendations.

## Overview

For a prescriptive approach, gather performance counters using PerfMon/LogMan and capture SQL Server wait statistics to better understand general pressures and potential bottlenecks of the source environment. 

Start by collecting the CPU, memory, [IOPS](/azure/virtual-machines/premium-storage-performance#iops), [throughput](/azure/virtual-machines/premium-storage-performance#throughput), and [latency](/azure/virtual-machines/premium-storage-performance#latency) of the source workload at peak times following the [application performance checklist](/azure/virtual-machines/premium-storage-performance#application-performance-requirements-checklist). 

Gather data during peak hours such as workloads during your typical business day, but also other high load processes such as end-of-day processing, and weekend ETL workloads. Consider scaling up your resources for atypically heavily workloads, such as end-of-quarter processing, and then scale done once the workload completes. 

Use the performance analysis to select the [VM Size](/azure/virtual-machines/sizes-memory) that can scale to your workload's performance requirements.


## Storage

SQL Server performance depends heavily on the I/O subsystem and storage performance is measured by IOPS and throughput. Unless your database fits into physical memory, SQL Server constantly brings database pages in and out of the buffer pool. The data files for SQL Server should be treated differently. Access to log files is sequential except when a transaction needs to be rolled back where data files, including `tempdb`, are randomly accessed. If you have a slow I/O subsystem, your users may experience performance issues such as slow response times and tasks that do not complete due to time-outs. 

The Azure Marketplace virtual machines have log files on a physical disk that is separate from the data files by default. The `tempdb` data files count and size meet best practices and are targeted to the ephemeral `D:\` drive. 

The following PerfMon counters can help validate the IO throughput required by your SQL Server: 
* **\LogicalDisk\Disk Reads/Sec** (read IOPS)
* **\LogicalDisk\Disk Writes/Sec** (write IOPS) 
* **\LogicalDisk\Disk Read Bytes/Sec** (read throughput requirements for the data, log, and `tempdb` files)
* **\LogicalDisk\Disk Write Bytes/Sec** (write throughput requirements for the data, log, and `tempdb` files)

Using IOPS and throughput requirements at peak levels, evaluate VM sizes that match the capacity from your measurements. 

If your workload requires 20K read IOPS and 10K write IOPS, you can either choose E16s_v3 (with up to 32K cached and 25600 uncached IOPS) or M16_s (with up to 20K cached and 10K uncached IOPS) with 2 P30 disks striped using Storage Spaces. 

Make sure to understand both throughput and IOPS requirements of the workload as VMs have different scale limits for IOPS and throughput.

## Memory

Track both external memory used by the OS as well as the memory used internally by SQL Server. Identifying pressure for either component will help size virtual machines and identify opportunities for tuning. 

The following PerfMon counters can help validate the memory health of a SQL Server virtual machine: 
* \Memory\Available MBytes
* [\SQLServer:Memory Manager\Target Server Memory (KB)](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)
* [\SQLServer:Memory Manager\Total Server Memory (KB)](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)
* [\SQLServer:Buffer Manager\Lazy writes/sec](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)
* [\SQLServer:Buffer Manager\Page life expectancy](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)

## Compute

Compute in Azure is managed differently than on-premises. On-premises servers are built to last several years without an upgrade due to the management overhead and cost of acquiring new hardware. Virtualization mitigates some of these issues but applications are optimized to take the most advantage of the underlying hardware, meaning any significant change to resource consumption requires rebalancing the entire physical environment. 

This is not a challenge in Azure where a new virtual machine on a different series of hardware, and even in a different region, is easy to achieve. 

In Azure, you want to take advantage of as much of the virtual machines resources as possible, therefore, Azure virtual machines should be configured to keep the average CPU as high as possible without impacting the workload. 

The following PerfMon counters can help validate the compute health of a SQL Server virtual machine:
* **\Processor Information(_Total)\% Processor Time**
* **\Process(sqlservr)\% Processor Time**

> [!NOTE] 
> Ideally, try to aim for using 80% of your compute, with peaks above 90% but not reaching 100% for any sustained period of time. Fundamentally, you only want to provision the compute the application needs and then plan to scale up or down as the business requires. 

## Connection Pooling Optimization

Connection pooling is an essential performance optimization technique supported by libraries such as ADO.NET and JDBC, which reduce the overhead associated with repeatedly opening and closing database connections. By reusing existing connections, applications can improve throughput, reduce latency, and minimize the resource demands on SQL Server running on Azure Virtual Machines. Efficient resource utilization is achieved by leveraging a cache of open database connections, significantly improving performance for applications with high-frequency or concurrent database access.

To optimize pooling, it is important to configure settings, such as `Max Pool Size`, in alignment with the available resources of the SQL Server instance and the expected workload. Over-provisioning the pool can lead to contention and resource exhaustion, while under-provisioning may result in connection delays. Libraries like ADO.NET and JDBC allow developers to control these settings through connection string parameters, enabling precise adjustments based on application requirements. Authentication mechanisms like Microsoft Entra ID (formerly Azure AD) can also impact connection pooling due to token expiration. Shorter connection lifetimes or token renewal mechanisms should be considered to prevent invalid connections from disrupting reuse.

Network latency and endpoint configurations can influence pooling efficiency. Public endpoints may introduce higher latency compared to private or direct connections, requiring adjustments to timeout settings and pool sizes. Proper firewall configurations and DNS settings are also critical to ensuring reliable connection reuse. In environments enforcing TLS/SSL encryption, connection strings must include encryption parameters, such as `Encrypt=True`, to prevent connection failures and maintain pooling effectiveness.

Monitoring the connection pool’s utilization and health is crucial for identifying bottlenecks or misconfigurations. Tools such as SQL Server Extended Events or application performance monitoring solutions can provide insights into connection behavior. Regularly optimizing pool settings based on workload patterns can help sustain high performance, especially in dynamic or cloud-native deployments. Properly configured, connection pooling using libraries like ADO.NET and JDBC can significantly enhance the performance and scalability of SQL Server on Azure Virtual Machines.

## Next steps

To learn more, see the other articles in this best practices series:

- [Quick checklist](performance-guidelines-best-practices-checklist.md)
- [VM size](performance-guidelines-best-practices-vm-size.md)
- [Storage](performance-guidelines-best-practices-storage.md)
- [Security](security-considerations-best-practices.md)
- [HADR settings](hadr-cluster-best-practices.md)


For security best practices, see [Security considerations for SQL Server on Azure Virtual Machines](security-considerations-best-practices.md).

Review other SQL Server Virtual Machine articles at [SQL Server on Azure Virtual Machines Overview](sql-server-on-azure-vm-iaas-what-is-overview.md). If you have questions about SQL Server virtual machines, see the [Frequently Asked Questions](frequently-asked-questions-faq.yml).
