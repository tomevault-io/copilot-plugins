## multi-tool-executer-mcp

> 1. install wasmedge or compile & install wasmedge if some additional feature has to be enabled

# the goal of project is to support python requests,sockets and FastMCP libraries to write MCP server which can execute python code.

## Detailed instruction to achieve the goal,
1. install wasmedge or compile & install wasmedge if some additional feature has to be enabled
2. compile and install RustPython with the python packages like requests,sockets and fastmcp
3. make use of webassembly virtual file system(VFS),create FastMCP server within virtual file system and FastMCP server should able to dynamically write python scripts and executes at runtime
4. when FastMCP server start with should create the UID for its instance
5. wasm program should either accept command line arguments or as environment variable with key as MCP_SERVERS_CONFIGS: Dict[Dict](input will be JSON string)
below is the vscode configuration
{
    "servers": {
        "multi_mcp_tool_executer": {
            "type": "stdio",
            "command": "wasmedge",
            "args": [
                "wasm_mcp_tool_executer.wasm",
            ],
            "env": {
                // below is MCP config for tool itself to call the tools internally
                "MCP_SERVERS_CONFIGS": "{
                    \"context7\": {
                        \"url\": \"https://mcp.context7.com/mcp\",
                        \"type\": \"http\"
                    },
                    \"chrome-mcp-server\": {
                        \"url\": \"http://127.0.0.1:12306/mcp\",
                        \"type\": \"http\"
                    },
                }"
            }
        },
    }
}
6. python script within wasm VFS will be running a FastMcp Server(fastmcp uses starlet to handle request),
    - FastMCP server will be up when its bootstrapped and it will take care of keep serving the request until server is stopped/killed from agent
    - which establish connection to provided MCP_SERVERS_CONFIGS using FastMCP client
    - will have single tool with name MultiToolExecuter with 2 parameters 1. python code 2. input which could be string,Dictionary,List etc and can return Any type,
    tool definition of MultiToolExecuter is to write a python code to a file in virtual file system, dynamically generate a UID for the instance and create a folder with the UID in the mounted folder, dynamically import the file and execute the function with input provided and return the result and result_explanation, like below,

    from fastmcp import FastMCP

    @mcp.tool(name="list_tools", description="This tool lists all the available tools in the MCP client."):
        def list_tools(fds_mcp_client: FastMCP) -> dict:
        tools = fds_mcp_client.list_tools()
        return """
            fds_mcp_client.list_components(Input type): output type
            fds_mcp_client.get_components(Input type): output type
            context7_mcp_client.research(Input type): output type
        """

    @mcp.tool(name="multi_tool_executer", description="""
        This tool takes python code as input and execute the tools,
        - provide only the definition of the function execute_tools
        - these code snippets should be able tu multiple tool call at once( and strictly input & output type for these tools should be - defined,So LLM can write a logic apply transformation anywhere in between
        -  python3 code should strictly have type annotations
        - the function should return a dictionary with 2 keys result and result_explanation

        here is the example code snippet for the function execute_tools
        `def execute_tools(initial_input:Any, **kwargs: Dict[str, FastMcp]) -> Map:
            // LLM python3 code start from here(below are a example code)

            fds_mcp_client=kwargs.get("fds_mcp_client")
            context7_mcp_client=kwargs.get("context7_mcp_client")

            structured_initial_input:MyStruct=initial_input
            stream_thinking_log("structured the output & all the necessary types are created")

            components_list=fds_mcp_client.list_components()
            stream_thinking_log("Listed all the components: count=${components_list.length}")

            required_components=components_list.filter(name=>name.include("card"))
            stream_thinking_log("Filtered required components : count=${required_components.length}")

            complete_documentations=required_components.map(name=>fds_mcp_client.get_components(name))
            stream_thinking_log("retrieved documentations for component")

            props_for_all_the_components=complete_documentations.map(comp=>({[comp.name]:comp.props}))
            stream_thinking_log("extracted props information from the documentation")

            return {
                result_explanation:"blah blah blah"
                result: props_for_all_the_components,
            }
        `
    """)
    def MultiToolExecuter(python_code:str,input:str) -> Dict:
        import importlib
        dynamic_module = importlib.import_module(instance_uid)
        return dynamic_module.execute_tools(input,**mcp_clients)



# Instructions for setup in order,
## Step1: Setup wasmedge to run RustPython code with additional libraries support
1. build everything using Dockerfile and try to keep all the setup in single Dockerfile with a multi stage build for efficient stage caching to reduce network latency
2. if necessary use MakeFile to bootstrap the setup
3. Docker should compile RustPython with all necessary packages to port it into wasmedge compatible wasm file
4. RustPython runtime should support additional libraries like requests,sockets and fastmcp, if its not support to install then you have to compile RustPython from source with these libraries and make sure to record the libraries and its version in a config/requirements.txt file, so whatever are in configuration file should be installed in the RustPython runtime and compiled to wasm after this
## Step2: Generate the wasm file using RustPython
1. you will be using RustPython runtime compiled to wasm with all the necessary library
2. the code written in python script will be included within wasm virtual file system and will be running a FastMcp Server as specified in the detailed instruction 
3. and the python code should be able to write files at runtime within virtual file system of wasm

## Step3: Make sure the `wasmedge wasm_mcp_tool_executer.wasm` should run as FastMCP server to support agent like, copilot and claude code
1. the mcp configuration looks like below,
{
    "servers": {
        "multi_mcp_tool_executer": {
            "type": "stdio",
            "command": "wasmedge",
            "args": [
                "wasm_mcp_tool_executer.wasm",
            ],
            "env": {
                // below is MCP config for tool itself to call the tools internally
                "MCP_SERVERS_CONFIGS": "{
                    \"context7\": {
                        \"url\": \"https://mcp.context7.com/mcp\",
                        \"type\": \"http\"
                    },
                    \"chrome-mcp-server\": {
                        \"url\": \"http://127.0.0.1:12306/mcp\",
                        \"type\": \"http\"
                    },
                }"
            }
        },
    }
}
or ,
if its possible embed wasmedge setup(for all supported platforms like Linux,Mac,Windows) & compiled wasm_mcp_tool_executer.wasm in uvx runtime(so it can be shipped as pip module .whl file) then you can use below configuration
{
    "servers": {
        "multi_mcp_tool_executer": {
            "type": "stdio",
            "command": "uvx",
            "args": [ 
                "wasm_mcp_tool_executer"
            ],
            "env": {
                // below is MCP config for tool itself to call the tools internally
                "MCP_SERVERS_CONFIGS": "{
                    \"context7\": {
                        \"url\": \"https://mcp.context7.com/mcp\",
                        \"type\": \"http\"
                    },
                    \"chrome-mcp-server\": {
                        \"url\": \"http://127.0.0.1:12306/mcp\",
                        \"type\": \"http\"
                    },
                }"
            }
        },
    }
}

# Finally,
- so Agent will directly execute wasmedge executable with wasm_mcp_tool_executer.wasm as input fle or it will execute bundled python package wasm_mcp_tool_executer which internally take care of execute wasmedge executable with wasm_mcp_tool_executer.wasm as input fle 


reference,
https://wasmedge.org/docs/develop/python/hello_world?_highlight=compile#compile-rustpython
https://github.com/RustPython/RustPython
https://github.com/webassemblylabs/webassembly-language-runtimes/tree/main/python
https://github.com/WebAssembly/wasi-sockets
https://gofastmcp.com/getting-started/quickstart

use context7 MCP to fetch documentations for planning and implementation


brief:
1. download and install wasmedge with socket support(don't build)
2. clone RustPython, add the required packages and compile/build RustPython+FastMCP server => wasm_mcp_tool_executer.wasm
3. and enable a virtual file system to dynamically write python script
4. remember this runtime will run FastMCP server on stdio transport, so nothing extra should be logged to console or standard input/output

(mostly it can't be packaged to pip package since wasmedge will directly runs wasm_mcp_tool_executer.wasm)


ultrathink

# don't use FDS mcp, confluence mcp for documentations for planning and implementation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brguru90)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brguru90)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
