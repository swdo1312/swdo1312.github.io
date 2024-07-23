---
layout: posts
title: GEOS-Chem 모델 모의 해보기
author: 도세원, 심창섭, 서정빈
categories: []
tags: [GEOS-Chem]
---



<p style="text-align:right">작성 : 도세원 (한국환경연구원, KEI)<br/> 
& 심창섭 (한국환경연구원, KEI)<br/>& 서정빈 (한국환경연구원, KEI)<br/>2024년 7월 23일</p>


---



## 소 개

[GEOS-Chem은 NASA Global Modeling and Assimilation Office](http://gmao.gsfc.nasa.gov/) 의 Goddard Earth Observing System(GEOS)에서 얻은 기상 입력을 통해 구동되는 대기 화학의 글로벌 3D 모델입니다 .  GEOS-Chem 모델을 통해 광범위한 대기오염물질을 분석할 수 있습니다.

모델의 개발 및 발전을 위하여 [GEOS-Chem Steering Committee](https://geoschem.github.io/steering-committee.html) 와 [Working Group](https://geoschem.github.io/working-groups.html)이 있으며, 이 모델은 은 US NASA Earth Science Division, the Canadian National and Engineering Research Council, and the Nanjing University of Information Sciences and Technology의 지원을 받아 하버드 대학교와 워싱턴 대학교에 있는 [GEOS-Chem 지원 팀](https://geoschem.github.io/support-team.html) 에서 관리합니다 .

자세한 정보와 링크는 [모델 개요 ](https://geoschem.github.io/overview.html) 참조.



GEOS-Chem과 관련된 주요한 소프트웨어는 5가지가 있다.

* **GEOS-Chem Classic** (이번 메뉴얼)
* [GCHP](https://gchp.readthedocs.io/) : 고해상도 버전(GEOS-Chem High Performance)
* [HEMCO](https://hemco.readthedocs.io/) : 배출량 정보
* [GCPy]([gcpy.readthedocs.io]) : Python toolkit
* [WRF-GC](http://wrf.geos-chem.org/) : GEOS-Chem in WRF
* [IMI](https://imi.readthedocs.io/) : Integrated Methane Inversion





## 하드웨어 요구사항

- 운영체제

  **GEOS-Chem Classic은 Unix와 유사한** 운영 체제가 있는 컴퓨터에서만 사용할 수 있다. 여기에는 모든 Linux 플레이버(예: Ubuntu, Fedora, Red-Hat, Rocky Linux, Alma Linux 등)와 BSD Unix(BSD 파생 버전인 MacOS X 포함)가 포함. 귀하의 기관에 컴퓨팅 리소스(예: 많은 코어가 있는 공유 컴퓨터 클러스터, 충분한 [디스크 스토리지](https://geos-chem.readthedocs.io/en/14.2.1/getting-started/disk-space.html#req-hard-disk) 및 [메모리](https://geos-chem.readthedocs.io/en/14.2.1/getting-started/memory.html#req-hard-mem) )가 있는 경우 GEOS-Chem Classic을 실행할 수 있다.

- RAM

  GEOS-Chem Classic 모델은 1개월을 수행하는데 벤치마크 시뮬레이션에서는 약 14GB의 메모리가 사용됨(모든 진단 변수를 저장할 경우). 이러한 이유는 진단 변수들이 모두 저장되기 때문이다. 그러나, 일반적인 상황에서는 모든 진단 변수를 출력하지 않기 때문에 벤치마크 시뮬레이션 보다는 훨씬 적은 메모리를 사용한다.

  - 4º × 5º 해상도 수행 :

    : 8~15GB

  - 2º × 2.5º 해상도 수행

    : 30~40GB 

  - 1º × 1.25º 또는 0.5º × 0.625º  등 고해상도 수행 

    : 최소 30GB 이상 

- 저장공간

  GEOS-Chem Classic에서 사용되는 데이터 형식은 NetCDF가 사용된다([netCDF 가이드](https://geos-chem.readthedocs.io/en/14.2.1/geos-chem-shared-docs/supplemental-guides/netcdf-guide.html#ncguide)). 대략적인 GEOS-Chem Classic 입력 데이터와 출력 데이터를 저장하는데 필요한 공간은 아래와 같다.

  - (입력자료) 

    MERRA-2 데이터 사용시

     :  1º × 1.25º   글로벌   ~30GB

     :  2º × 2.5º   글로벌   ~110GB

     :  0.5º × 0.625º   중앙아시아(AS)   ~115GB

     :  0.5º × 0.625º   유럽(EU)   ~58GB

     :  0.5º × 0.625º   북미(NA)   ~110GB

  - (출력자료) 

    [GEOS-Chem 13.0.0 벤치마크 결과](https://wiki.geos-chem.org/GEOS-Chem_13.0.0#GEOS-Chem_Classic_13.0.0_vs_12.9.0)에 따르면,1개월을 수행한 결과 837MB의 용량이 발생하였다. 이 중에서 진단변수는 ~646MB를 차지하였고, 재시작 파일은 ~191MB를 차지하였다. 공간을 절약하기 위한 자세한 내용은 [GEOS-Chem History 진단](https://geos-chem.readthedocs.io/en/14.2.1/gcclassic-user-guide/history-diag-ref.html#history-diagnostics) 섹션을 참조. 

    (참고) GEOS-Chem 13.0.0부터 코드 수정을 통해 진단 배열이 필요한 출력을 저장하는 데 필요한 충분한 요소로만 차원이 지정되도록 수정되었다. 예를 들어 SpeciesConc_O3 진단만 출력하려는 경우 GEOS-Chem은 관련 배열을 (NX, NY, NZ, 1) 요소(1개 종만 보관하기 때문에 1)로 차원을 지정이 된다. 이를 통해 시뮬레이션에 필요한 메모리 양을 크게 줄일 수 있습니다.

    

    

## 소프트웨어 요구사항

사용하는 소프트웨어에 따라 다양한 상황에 처할 수 있고 설명이 복잡해 지므로, 아래를 참고하길 바란다.

- [컴파일러](https://geos-chem.readthedocs.io/en/latest/getting-started/compilers.html)
  - intel
  - GNU

* [필수 소프트웨어](https://geos-chem.readthedocs.io/en/latest/getting-started/packages-req.html)

  - Git

  - Cmake (3.13 이상)

  - GNU Make (GEOS-Chem 13.0.0 이상에선 필요 없음)

  - NetCDF

- [권장되는 소프트웨어](https://geos-chem.readthedocs.io/en/latest/getting-started/packages-opt.html)

  - GCpy

  - gdb and cgdb

  - ncview

  - nco

  - cdo

  - KPP

  - flex and bison

    

## 1. 모델 및 자료 가져오기

GEOS-Chem 모델을 사용하기 위해서는 모델 소스코를 다운로드가 필요하다.

자세한 설명은  [여기](https://geos-chem.readthedocs.io/en/14.2.1/getting-started/quick-start.html)를 참조.

* GEOS-Chem 모델 다운로드

  ``` bash
  cd ~/MODL/geoschem/
  git clone --recurse-submodules https://github.com/geoschem/GCClassic.git GCClassic
  ```

  참고) 원하는 경우 폴더 이름을 수정할 수 있음

  ```
  git clone --recurse-submodules https://github.com/geoschem/GCClassic.git my_code_dir
  ```



저자의 경우 다운로드 된 GCClassic 버전에 따라 폴더를 분리하기 위하여 추가적인 폴더 정리를 하였음. GCClassic/CHANGELOG.md 파일 안에 기록된 버전에 따라 ~/MODL/geoschem/Code.14.4.1/ 를 생성하여 다운로드 된 GCClassic/ 파일을 이동하였음. (2024.07.19기준 최신버전, v14.4.1)



## 2. 실행 디렉토리 생성

GEOS-Chem을 수행하기 위해 컴파일 할 디렉토리를 성정하여야 한다. 앞으로 여러 실험을 수행할 것을 대비하여 rundirs/ 폴더를 만들고 그 안에 모델을 컴파일 하였다. 



``` bash
mkdir ~/MODL/geoschem/ExtData/
mkdir ~/MODL/geoschem/Code.14.4.1/rundirs
cd ~/MODL/geoschem/Code.14.4.1/GCClassic/run
./createRunDir.sh
```



참고) 현재 사용중인 서버 계정에서 GEOS-Chem 을 처음 수행할 경우 각 계정 안 ./geoschem/config 파일을 생성하기 위하여 몇가지 답변이 필요하다. 아래에 질문사항에 따라 답변을 한다면 큰 어려움이 없다.

``` bash
-----------------------------------------------------------
Enter path for ExtData:  (각자 서버에서 처음 geoschem 모델을 수행시 ./geoschem/config 를 생성하게 된다. [입력자료를 저장한 공간 설정] )
-----------------------------------------------------------   
>>> ~/MODL/geoschem/ExtData/

Initiating User Registration:
You will only need to fill this information out once.
Please respond to all questions.  Registration is required
in order to receive GEOS-Chem support.

-----------------------------------------------------------
What is your name (first and last)?  (./geoschem/comfig 에 저장되는 모델 사용자에 대한 Registration 등록 과정)
What is your email address?
What is the name of your research institution?
What is the name of your principal invesigator?
Please provide the web site for your institution
Please provide your github username 
Where do you plan to run GEOS-Chem?

Successful Registration
-----------------------------------------------------------
```





GEOS-Chem 모델을 수행하기 위한 몇가지 옵션을 선택하여야 한다. 이번 튜토리얼에서는 수행하는 옵션은 아래와 같다. 정상적으로 수행되었다면, 옵션에서 선택한 값들이 사용된 폴더명이 생성될 것이다.

예시) gc_4x5_47L_merra2_CH4

``` bash
-----------------------------------------------------------
Choose simulation type:
-----------------------------------------------------------
   1. Full chemistry
   2. Aerosols only
   3. Carbon
   4. Hg
   5. POPs
   6. Tagged O3
   7. TransportTracers
   8. Trace metals
   9. CH4
  10. CO2
  11. Tagged CO
>>> 9

-----------------------------------------------------------
Choose meteorology source:
-----------------------------------------------------------
  1. MERRA-2 (Recommended)
  2. GEOS-FP
  3. GEOS-IT (Beta release)
  4. GISS ModelE2.1 (GCAP 2.0)
>>> 1

-----------------------------------------------------------
Choose horizontal resolution:
-----------------------------------------------------------
  1. 4.0  x 5.0
  2. 2.0  x 2.5
  3. 0.5  x 0.625
>>> 2

-----------------------------------------------------------
Choose number of levels:
-----------------------------------------------------------
  1. 72 (native)
  2. 47 (reduced)
>>> 2

-----------------------------------------------------------
Enter path where the run directory will be created:
-----------------------------------------------------------
>>> ~/MODL/geoschem/Code.14.4.1/rundirs
Expanding to ~/MODL/geoschem/Code.14.4.1/rundirs

-----------------------------------------------------------
Enter run directory name, or press return to use default:

NOTE: This will be a subfolder of the path you entered above.
-----------------------------------------------------------
>>>
  -- Using default directory name gc_2x25_47L_merra2_CH4
  -- See rundir_vars.txt for summary of default run directory settings
  -- This run directory has been set up to start on 20190101
  -- A restart file for this date has been copied to the Restarts subdirectory
  -- You may add more restart files using format GEOSChem.Restart.YYYYMMDD_HHmmz.nc4
  -- Change simulation start and end dates in configuration file geoschem_config.yml
  -- Default frequency and duration of diagnostics are set to monthly
  -- Modify diagnostic settings in HISTORY.rc and HEMCO_Config.rc

  -- The following sample restart provided for this simulation was not found:
     /home/swdo02/models/geoschem/ExtData/GEOSCHEM_RESTARTS/v2020-02/GEOSChem.Restart.CH4.20190101_0000z.nc4
     You will need to provide this initial restart file or disable
     GC_RESTARTS in HEMCO_Config.rc to initialize your simulation
     with default background species concentrations.

-----------------------------------------------------------
Do you want to track run directory changes with git? (y/n)
-----------------------------------------------------------
>>> n


Created  ~/MODL/geoschem/Code.14.4.1/rundirs/gc_2x25_47L_merra2_CH4
```





## 3. 빌드 구성 및 컴파일

GEOS-Chem은 실행 디렉토리 또는 시스템의 다른 곳에서 빌드할 수 있다. 그러나, 실행 디렉토리에서 GEOS-Chem을 빌드하는 것을 추천한다.  실행 디렉토리에서 빌드를 수행할 경우 모든 빌드 파일이 모델을 실행할 위치와 가까운 곳에 보관되므로 편리하게 사용할 수 있다.

``` bash
cd ~/MODL/geoschem/Code.14.4.1/rundirs/gc_2x25_47L_merra2_CH4/build
cmake ../CodeDir -DRUNDIR=..
make -j
make install
```

위 과정을 따라서 컴파일에 성공하였다면 /build/bin/gcclassic 파일이 생성된다.







## 모든 실험을 위한 기본 사항

GEOS-Chem 모델을 수행하기 위한 빌드 및 컴파일이 끝났다면, 이제는 모델 수행에 필요한 옵션을 설정하여야 한다. 모델을 수행하기 전 아래의 파일에 대한 설정을 검토하여야 한다. [옵션 가이즈 참조](https://geos-chem.readthedocs.io/en/14.2.1/geos-chem-shared-docs/supplemental-guides/customize-guide.html#customguide)



* README : 코드 및 모델 설정 및 실행 방법에 대한 유용한 정보가 들어 있음

*  [geoschem_config.yml](https://geos-chem.readthedocs.io/en/14.2.1/gcclassic-user-guide/geoschem-config.html#cfg-gc-yml) : 실험에 대한 각종 설정(시작 및 종료 시간, 켜거나 끌 작업 등) 수정.

* [HISTORY.rc](https://geos-chem.readthedocs.io/en/14.2.1/gcclassic-user-guide/history.html#cfg-hist) : GEOS-Chem 진단 변수에 대한 설정.

* [HEMCO_Diagn.rc](https://geos-chem.readthedocs.io/en/14.2.1/gcclassic-user-guide/hemco-diagn.html#cfg-hco-diagn) : HEMCO 통해 배출량에 대한 설정.

* [HEMCO_Config.rc](https://geos-chem.readthedocs.io/en/14.2.1/gcclassic-user-guide/hemco-config.html#cfg-hco-cfg) : HEMCO 통해 저장공간(~/MODL/geoschem/ExtData/) 에서 어떤 배출량 인벤토리를 읽어들일지 설정.



## 4. geoschem_config.yml

이 파일은 모델을 수행하는데 필요한 옵션을 설정하는 텍스트 기반의 파일이다(GEOS-Chem 14.0.0부터 input.geos 대체됨).

* 4.1 모델 실험기간 설정

  ```
  ### geoschem_config.yml: GEOS-Chem Runtime configuration options.
  ### Customized for simulations using the CH4 mechanism.
  
  #============================================================================
  # Simulation settings
  #============================================================================
  simulation:                                                             # GEOS-Chem 시뮬레이션 유형 설정, 'fullchem','aerosol'
    name: CH4                                                             # 'carbon','CH4','CO2','Hg','POPs','TransprotTracers'
    start_date: [20190101, 000000]                                        # 시작시간 설정 [YYYYMMDD, hhmmss]
    end_date: [20190201, 000000]                                          # 끝시간 설정 [YYYYMMDD, hhmmss]
    root_data_dir: /home/swdo/MODL/geoschem/gcgrid/data/ExtData           # root 경로, HEMCO 등 입력자료가 저장된 위치
    met_field: MERRA2                                                     # GEOS-Chem 기상입력자료 : 'MERRA2','GEOS-FP','FCPA2'
    species_database_file: ./species_database.yml                         # GEOS-Chem Species Database 파일 경로
    species_metadata_output_file: OutputDir/geoschem_species_metadata.yml #  
    verbose:                                                              # LOG 출력을 설정
      activate: false                                                     # activate : log 정보를 활성화(true), 비활성화(false)
      on_cores: root       # Allowed values: root all                     # on_cores : 어떤 노드('root','all')에 대한 정보 출력 
    use_gcclassic_timers: false                                           # GEOS-Chem 수행에 걸리는 시간 출력할지 설정
  
  ```

* 4.2 그리드 설정

  ```
  #============================================================================
  # Grid settings
  #============================================================================
  grid:
    resolution: 2.0x2.5                                                   # 수평해상도 설정 : 4º×5º, 2.0º×2.5º, 0.5º×0.625º
    number_of_levels: 72                                                  # 수직해상도 설정 : 72, 47, 40(GCAP2) 
    longitude:                                                            
      range: [-180.0, 180.0]                                              # 위도의 범위 설정 [최소, 최대]
      center_at_180: true                                                 # true 일 경우, 
    latitude:                                                             
      range: [-90.0, 90.0]                                                # 위도의 범위 설정 [최소, 최대]
      half_size_polar_boxes: true                                         # ture 일 경우,
    nested_grid_simulation:                                               # nesting 그리드 사용에 대한 옵션 
      activate: false                                                     # true : nesting, false : 단일 grid
      buffer_zone_NSEW: [0, 0, 0, 0]                                      # nesting grid에서 buffer zone 영역 설정 (3,3,3,3) 추천
      
  ```

- 4.3 모델 적분 시간 설정

  ```
  #============================================================================
  # Timesteps settings
  #============================================================================
  timesteps:                                                              # 모델의 적분시간 간격 설정
    transport_timestep_in_s: 600                                          # 대기수송모델 계산 간격 전역(600), nesting(300)
    chemistry_timestep_in_s: 1200                                         # 화학반응 및 배출 간격: (추천) 전역(1200), nesting(600)
    # radiation_timestep_in_s                                             # fullchem 사용시 RRTMG 복사모델 계산 간격
  
  ```

- 4.4 변수 입력 설정

  ```
  #============================================================================
  # Settings for GEOS-Chem operations
  #============================================================================
  operations:
    chemistry:                                                           
      activate: true                                                     # GEOS-Chem 에서 화학반응 활성화(true), 비활성화(false)
    convection:                                                          
      activate: true                                                     # GEOS-Chem 에서 구름 대류 활성화(true), 비활성화(false)
    pbl_mixing:                                                           
       activate: true                                                    # GEOS-Chem 에서 PBL 혼합 활성화(true), 비활성화(false)
       use_non_local_pbl: true                                           # non-local PBL mixing scheme (VDIFF) 사용 유무
  
    transport:
      gcclassic_tpcore:                 # GEOS-Chem Classic only         # 대류를 통한 물질 이동 제어 옵션
        activate: true                  # GEOS-Chem Classic only         # 활성화(true), 비활성화(false)
        fill_negative_values: true      # GEOS-Chem Classic only         # true : 음수일 경우 농도=0, false : 농도변화 없음
        iord_jord_kord: [3, 3, 7]       # GEOS-Chem Classic only         # 대류옵션 지정 
                                                                         # 1. 1st order upstream scheme, 2.2nd order van Lee
                                                                         # 3. Monotonic PPM, 4.Semi-monotonic PPM
                                                                         # 5.Positive-definite PPM, 6.Un-constrained PPM
                                                                         # 7.Huynh/Van Leer/Lin full monotonicity constrain
                                                                         # 권장옵션 [3, 3, 7]
      transported_species:
        - CH4                                                            # 대류를 통해 이동되는 물질
  
  #============================================================================
  # Options for CH4
  #============================================================================
  CH4_simulation_options:
  
    use_observational_operators:                                         # CH4 관측에 관한 옵션
      AIRS: false                                                        # 비행기 관측 사용 유무, [defalte, false]
      GOSAT: false                                                       # GOSAT 관측 사용 유무, [defalte, false]
      TCCON: false                                                       # TCCON 관측 사용 유무, [defalte, false]
  
    analytical_inversion:
      perturb_CH4_boundary_conditions: false                             # nesting 사용시 경계조건에서 교란 활성화(true)
      CH4_boundary_condition_ppb_increase_NSEW: [0.0, 0.0, 0.0, 0.0]     # nesting 사용시 경계조건에서 교란량(ppbv) 설정
  
  
  ```



- 4.5 추가적인 변수 입력 설정

  ```
  #============================================================================
  # Settings for diagnostics (other than HISTORY and HEMCO)
  #============================================================================
  extra_diagnostics:
  
    obspack:                                                             
      activate: false                                                    # ObsPack 자료를 추가적으로 사용할 경우(true)
      quiet_logfile_output: false                                        # log 파일 생성시 (true)
      input_file: ./obspack_co2_1_OCO2MIP_2018-11-28.YYYYMMDD.nc         # ObsPack 파일의 경로 설정
      output_file: ./OutputDir/GEOSChem.ObsPack.YYYYMMDD_hhmmz.nc4       # 
      output_species:                                                    # 출력 파일에 사용되는 변수
        - CH4
  
    planeflight:
      activate: false                                                    # 비행기 관측 자료를 추가적으로 사용할 경우 (true) 
      flight_track_file: Planeflight.dat.YYYYMMDD                        # 비행기 관측 자료 경로 설정
      output_file: plane.log.YYYYMMDD
  
  ```













## 5. HISTORY.rc

GEOS-Chem 모델을 수행을 통해 계산된 진단 변수들에 대한 저장을 설정하는 파일이다.



- 모델결과 파일명 및 저장위치 설정

  ```
  ###############################################################################
  ###  HISTORY.rc file for GEOS-Chem CH4 specialty simulations                ###
  ###  Contact: GEOS-Chem Support Team (geos-chem-support@g.harvard.edu)      ###
  ###############################################################################
  
  EXPID:  ./OutputDir/GEOSChem                             # GEOS-Chem 모델 결과 파일명 설정
                                                           # ex) EXPID: ./Hello 설정시 
                                                           # ~/gc_2x25_47L_merra2_CH4/Hello* 파일명으로 된 결과 파일 생성
                                                       
  #==============================================================================
  # %%%%% COLLECTION NAME DECLARATIONS %%%%%
  #
  # To enable a collection, remove the "#" character in front of its name. The
  # Restart collection should always be turned on.
  #
  # NOTE: These are the "default" collections for GEOS-Chem, but you can create
  # your own customized diagnostic collections as well.
  #==============================================================================
  COLLECTIONS: 'Restart',                                  # 저장할 변수들 설정
               'CH4',                                      # 파일명 : ${EXPID}.'Restart'*.nc4 or ${EXPID}.'CH4'*.nc4 
               'Metrics',
               'SpeciesConc',
               #'Budget',
               #'CloudConvFlux',
               #'ConcAfterChem',
               #'LevelEdgeDiags',
               #'SatDiagn',
               #'SatDiagnEdge',
               'StateMet',
               #'BoundaryConditions',
  ::
  ```



- 기본적인 옵션

  ```
  #==============================================================================
  # %%%%% THE SpeciesConc COLLECTION %%%%%
  #
  # GEOS-Chem species concentrations (default = all species)
  #
  # Available for all simulations
  #
  # Concentrations may be saved out as SpeciesConcVV  [v/v dry air] or
  #                                    SpeciesConcMND [molec/cm3]
  #==============================================================================
    SpeciesConc.template:       '%y4%m2%d2_%h2%n2z.nc4',                 # 파일명 날짜 형식 지정 : YYYYMMDD_hhmmz.nc4
    SpeciesConc.frequency:      00000100 000000                          # 저장되는 시간 간격 설정 [YYYYMMDD, hhmmss]
                                                                         # (ex) 00000000 01000 매 시간 저장
                                                                         #      00000100 00000 매 달 저장
    SpeciesConc.duration:       00000100 000000                          # 새로운 netCDF 파일을 얼마나 자주 생성할지
                                                                         # (ex) 00000000 01000 매 시간 저장
                                                                         #      00000001 00000 매일 새로운 netCDF 파일 생성       
    SpeciesConc.mode:           'time-averaged'                          # 저장하는 방법 설정
                                                                         # 'instantaneous' : 스냅샷, 'time-averaged' : 시간 평균
    SpeciesConc.fields:         'SpeciesConcVV_?ALL?           ',        # 저장되는 변수 목록 설정 
                                ##'SpeciesConcMND_?ALL?          ',      # ?ADA? : Only the advected species
                                                                         # ?AER? : Only the aerosol species
                                                                         # ?ALL? : All GEOS-Chem species
                                                                         # ?DRYALT? : Only the dry-deposited species whose 
                                                                                      concentrations we wish to archive at a 
                                                                                      given altitude above the surface. (In 
                                                                                      practice these are only O3 and HNO3.)
                                                                         # ?DRY? : Only the dry-deposited species
                                                                         # ?FIX? : Only the inactive (aka “fixed”) species in 
                                                                                   the KPP chemical mechanis
                                                                         # ?GAS? : Only the gas-phase species
                                                                         # ?HYG? : Only aerosols that undergo hygroscopic 
                                                                                   growth (sulfate, BC, OC, SALA, SALC)
                                                                         # ?LOS? : Only chemical loss species or families
                                                                         # ?KPP? : Only the KPP species
                                                                         # ?VAR? : Only the photolyzed species
                                                                         # ?WET? : Only the wet-deposited species
                                                                         # ?PRD? : Only chemical production species or families
                                                                         # ?DUSTBIN? : Only the dust bin number
                                                                         # ?PHOTOBIN? : Number of a given wavelength bin for 
                                                                                        FAST-JX photolysis                     
  ::
  #==============================================================================
  
  ```







## 6. HEMCO_config.rc 및 HEMCO_Diagn.rc 

GEOS-Chem Classic은 배출 및 플럭스를 계산하기 위해서 [HEMCO](https://hemco.readthedocs.io/)에 의존한다. HEMCO에 대한 설정은 [구성파일(HEMCO_Config.rc)](https://hemco.readthedocs.io/en/latest/hco-ref-guide/hemco-config.html)에서 수정을 통해 변경이 가능하다. 



- 기본설정 (HEMCO_config.rc)

  ```
  ###############################################################################
  ### BEGIN SECTION SETTINGS
  ###############################################################################
  
  ROOT:                        ~/MODL/geoschem/gcgrid/data/ExtData/HEMCO
  GCAPSCENARIO:                not_used
  GCAPVERTRES:                 47
  Logfile:                     *
  DiagnFile:                   HEMCO_Diagn.rc
  DiagnPrefix:                 ./OutputDir/HEMCO_diagnostics
  DiagnFreq:                   Monthly
  Wildcard:                    *
  Separator:                   /
  Unit tolerance:              1
  Negative values:             2
  Only unitless scale factors: false
  Verbose:                     false
  VerboseOnCores:              root       # Accepted values: root all
  
  ### END SECTION SETTINGS ###
  
  ```



- 배출 인벤토리 설정 (HEMCO_config.rc)

  ```
  ###############################################################################
  ### BEGIN SECTION EXTENSION SWITCHES
  ###############################################################################
  # ExtNr ExtName                on/off  Species   Years avail.
  0       Base                   : on    *
  # ----- MAIN SWITCHES ---------------------------------------------------------
      --> EMISSIONS              :       true
      --> METEOROLOGY            :       true      # 1980-2021
      --> CHEMISTRY_INPUT        :       true
  # ----- RESTART FIELDS --------------------------------------------------------
      --> GC_RESTART             :       true
  # ----- NESTED GRID FIELDS ----------------------------------------------------
      --> GC_BCs                 :       false
  # ----- REGIONAL INVENTORIES --------------------------------------------------
      --> GHGI_v2                :       false    # 2012-2018
      --> GHGI_v2_Express_Ext    :       true     # 2012-2020
      --> Scarpelli_Canada       :       true     # 2018
      --> Scarpelli_Mexico       :       true     # 2015
  # ----- GLOBAL INVENTORIES ----------------------------------------------------
      --> GFEIv2                 :       true     # 2019
      --> EDGARv8                :       true     # 2010-2022
      --> QFED2                  :       false    # 2009-2015
      --> JPL_WETCHARTS          :       true     # 2009-2017
      --> SEEPS                  :       true     # 2012
      --> LAKES                  :       false    # 2009-2015
      --> RESERVOIRS             :       true     # 2022
      --> FUNG_TERMITES          :       true     # 1985
      --> FUNG_SOIL_ABSORPTION   :       false    # 2009-2015
      --> MeMo_SOIL_ABSORPTION   :       true     # 1990-2009 or clim.
  # ----- FUTURE EMISSIONS ------------------------------------------------------
      --> RCP_3PD                :       false    # 2005-2100
      --> RCP_45                 :       false    # 2005-2100
      --> RCP_60                 :       false    # 2005-2100
      --> RCP_85                 :       false    # 2005-2100
  # ----- CMIP6 ANTHRO EMISSIONS ------------------------------------------------
  #   Set GCAPSCENARIO (e.g., HIST, SSP585) above in SECTION SETTINGS
      --> CMIP6_SFC_LAND_ANTHRO  :       false    # 1850-2100
      --> CMIP6_SHIP             :       false    # 1850-2100
      --> BB4MIPS                :       false    # 1850-2100
  # ----- NON-EMISSIONS DATA ----------------------------------------------------
      --> CH4_LOSS_FREQ          :       true     # 1985
      --> GLOBAL_OH              :       true     # 2010-2019
      --> GLOBAL_CL              :       true     # 2010-2019
      --> OLSON_LANDMAP          :       true     # 1985
      --> YUAN_MODIS_LAI         :       true     # 2000-2020
  # ----- OPTIONS FOR ANALYTICAL INVERSIONS  ------------------------------------
      --> AnalyticalInversion    :       false
      --> UseTotalPriorEmis      :       false    # Skips global/regional inventories
      --> Emis_PosteriorSF       :       false    # Apply posterior scale factors to total emis?
      --> OH_PosteriorSF         :       false    # Apply posterior scale factor  to global OH?
  # -----------------------------------------------------------------------------
  111     GFED                   : on    CH4
      --> GFED4                  :       true
      --> GFED_daily             :       true
      --> GFED_3hourly           :       false
      --> Scaling_CO             :       1.05
      --> Scaling_NAP            :       2.75e-4
      --> hydrophilic BC         :       0.2
      --> hydrophilic OC         :       0.5
      --> fraction POG1          :       0.49
  114     FINN                   : off   CH4
      --> FINN_daily             :       true
      --> Scaling_CO             :       1.0
      --> hydrophilic BC         :       0.2
      --> hydrophilic OC         :       0.5
  
  ### END SECTION EXTENSION SWITCHES ###
  
  ###############################################################################
  ### BEGIN SECTION BASE EMISSIONS
  ###############################################################################
  
  
  ```







- 배출 인벤토리 설정 (HEMCO_Diagn.rc)

  ```
  #------------------------------------------------------------------------------
  #                  GEOS-Chem Global Chemical Transport Model                  !
  #------------------------------------------------------------------------------
  #BOP
  #
  # !MODULE: HEMCO_Diagn.rc
  #
  # !DESCRIPTION: Configuration file for netCDF diagnostic output from HEMCO.
  #\\
  #\\
  # !REMARKS:
  #  Customized for the CH4 simulation.
  #
  # !REVISION HISTORY:
  #  18 Oct 2018 - R. Yantosca - Added comment header and longname metadata.
  #                              Also changed output unit to kg/m2/s.
  #EOP
  #------------------------------------------------------------------------------
  #BOC
  # Name               Spec ExtNr Cat Hier Dim  OutUnit  LongName
  EmisCH4_Total        CH4   -1   -1   -1   2   kg/m2/s  CH4_emissions_from_all_sectors
  EmisCH4_Oil          CH4    0    1   -1   2   kg/m2/s  CH4_emissions_from_oil
  EmisCH4_Gas          CH4    0    2   -1   2   kg/m2/s  CH4_emissions_from_gas
  EmisCH4_Coal         CH4    0    3   -1   2   kg/m2/s  CH4_emissions_from_coal
  EmisCH4_Livestock    CH4    0    4   -1   2   kg/m2/s  CH4_emissions_from_livestock
  EmisCH4_Landfills    CH4    0    5   -1   2   kg/m2/s  CH4_emissions_from_landfills
  EmisCH4_Wastewater   CH4    0    6   -1   2   kg/m2/s  CH4_emissions_from_wastewater
  EmisCH4_Rice         CH4    0    7   -1   2   kg/m2/s  CH4_emissions_from_rice
  EmisCH4_OtherAnth    CH4    0    8   -1   2   kg/m2/s  CH4_emissions_from_other_anthropogenic_sources
  EmisCH4_BiomassBurn  CH4  111   -1   -1   2   kg/m2/s  CH4_emissions_from_biomass_burning
  EmisCH4_Wetlands     CH4    0   10   -1   2   kg/m2/s  CH4_emissions_from_wetlands
  EmisCH4_Seeps        CH4    0   11   -1   2   kg/m2/s  CH4_emissions_from_geological_seeps
  EmisCH4_Lakes        CH4    0   12   -1   2   kg/m2/s  CH4_emissions_from_lakes
  EmisCH4_Termites     CH4    0   13   -1   2   kg/m2/s  CH4_emissions_from_termites
  EmisCH4_SoilAbsorb   CH4    0   14   -1   2   kg/m2/s  CH4_emissions_from_soil_absorption
  EmisCH4_Reservoirs   CH4    0   15   -1   2   kg/m2/s  CH4_emissions_from_hydroelectric_reservoirs
  
  #EOC
  
  
  # Name : netCDF variable name for the requested diagnostic quantity
  # Spec : Species name
  # ExtNr : Extension number (-1 means sum over all extensions)
  # Cat : Category (-1 means sum over all categories)
  # Hier : Hierarchy (-1 means sum over all hierarchies)
  # Dim : Dimension of data (1: scalar, 2: lon-lat, 3: lon-lat-lev)
  # OutUnit  : Units
  # LongName : Value for the long_name netCDF variable attribute
  ```





모든 실험에 대한 옵션 설정이 끝났다면, GEOS-Chem을 실행하게 된다.





## 7. GEOS-Chem Classic 실행

모든 실험에 대한 옵션 설정이 끝났다면, GEOS-Chem Classic을 실행을 위한 준비가 끝났다. 이제는 수행기간에 따른 HEMCO 자료를 다운 받고 모델을 수행하면 된다. 모델을 dryrun 옵션을 사용하여 수행한다.



GEOS-Chem Classic 모의 순서는 :

- 모델 수행 기간 HEMCO 데이터 유무 확인 (./gcclassic --dryrun)
- 모델 수행 기간 부족한 HEMCO 데이터 다운로드 (./dowuload_data.py)
- 모델 수행 (./gcclassic)





### 7-1. “HEMCO 데이터 유무 확인” 

아래와 같이 --dryrun 명령어를 사용하여 GEOS-Chem Classic 실행 파일을 실행한다. 여기서 " | tee" 옵션을 통해 출력 결과를 log.dryrun 에 저장한다.

``` bash
./gcclassic --dryrun |tee log.dryrun
```

수행된 결과를 확인해보면 일반적인 GEOS-Chem 로그 파일과 다소 비슷해 보이지만 데이터 파일 목록과 각 파일이 디스크에서 저장이되어 있는지를 포함하고 있다. 이 정보를 바탕으로 다운로드가 되지 않은 파일은 다음 단계에서 다운로드 받게 된다.

(참고) log.dryrun 파일명은 수정하여 사용하여도 된다. 그러나, 모델 수행기간에 필요한 HEMCO 데이터에 대한 정보가 이 log 파일에 포함되어 있으니 다음 단계에서도 동일한 log 파일명을 사용하여야 한다.



### 7-2. “부족한 HEMCO 데이터 다운로드” 

다운로드 되지 않은 HEMCO 파일의 정보는 log.dryrun 에 기록되어 있다. 관련 데이터를 다운로드 받을 수 있는 몇가지 경로가 있지만 WashU(세인트루이스의 워싱턴 대학교) 사이트 ( [http://geoschemdata.wustl.edu](http://geoschemdata.wustl.edu/) )에서 데이터를 다운로드하는 것을 추천한다. WashU의 Randall Martin 그룹에서 관리하는 GEOS-Chem의 주요 데이터 사이트이다.

``` bash
./dowuload_data.py log.dryrun wu

(참고) ./download data.py log.dryrun amazon
      ./download data.py log.dryrun rochester
```

-  (참고) 세인트루이스의 워싱턴 대학교 사이트 이외로는 "Amazon Web Services 클라우드", "atmos.earth.rochester.edu 사이트(Rochester)" 가 있다.

  ``` bash
  ./download data.py log.dryrun amazon     # GEOS-Chem에 필요한 데이터를 빠르게 다운로드 할 수 있다.
  ./download data.py log.dryrun rochester  # GCAP 2.0 기상 현장 데이터, 이 기상 현장 데이터는 산업화 이전 or 미래로 시뮬레이션 수행시 유용
  ```


-  (참고) 다운로드를 하지 않고 건너뛰고 싶을 때

  ``` bash
  ./download_data.py log.dryrun --skip-download   # 이미 모델은 수행했지만 문서화와 같은 목적으로 파일을 생성하고자 할 때
  ```

  

### 7-3. “모델 수행 (./gcclassic)” 

``` bash
./gcclasic > log.run
```

모델이 정상적으로 수행되었다면 log.run 파일 맨 밑에 "E N D   O F   G E O S -- C H E M" 을 확인할 수 있을 것이다. 모델 결과는 HISTORY.rc 에서 설정한 경로에 따라 NetCDF 파일(예: `OutputDir/GEOSChem*.nc4`및 `OutputDir/HEMCO*.nc`)로 저장되어 있을 것이다. 이 결과 파일들은 netCDF 형식으로 만들어진 것이므로 [ncview](http://meteora.ucsd.edu/~pierce/ncview_home_page.html) 등으로 간단히 확인할 수 있습니다.







## 8. 모의 결과 분석

GEOS-Chem 모델을 시각화 하는데 GCPy 프로그램이 많이 사용되고 있다. GCPy 는 Python toolkit 으로 누구나 손 쉽게 사용할 수 있는 장점이 있다. 이와 관련된 내용은 새로운 포스팅에서 소개하고자 한다. 



<center>수고하셨습니다! ^-^.</center>



<p style="text-align:right">작성 : 도세원 (한국환경연구원, KEI) <br/> 
& 심창섭 (한국환경연구원, KEI)<br/>& 서정빈 (한국환경연구원, KEI)<br/>
    2024년 7월 23일</p>


---

<center>- END -</center>

---

 

 

 
