
import React, { useState, useRef, useEffect } from 'react';
import { 
  Send, Paperclip, Mic, Image as ImageIcon, 
  Menu, Plus, MessageSquare, MoreHorizontal, 
  Bot, User, Sparkles, Code, Terminal, X,
  ChevronRight, ChevronDown, Loader2
} from 'lucide-react';
import { motion, AnimatePresence } from 'framer-motion';
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

// Utility for clean class merging
function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// --- Types ---
type MessageType = 'user' | 'ai';
type ToolStatus = 'idle' | 'running' | 'completed' | 'failed';

interface ToolInvocation {
  id: string;
  toolName: string;
  status: ToolStatus;
  input: any;
  output?: any;
}

interface Message {
  id: string;
  type: MessageType;
  content: string;
  timestamp: Date;
  tools?: ToolInvocation[]; // AI messages might have attached tool calls
}

// --- Components ---

/**
 * 1. Sidebar Component
 * Collapsible history navigation similar to Gemini/ChatGPT
 */
const Sidebar = ({ isOpen, toggle }: { isOpen: boolean; toggle: () => void }) => {
  return (
    <motion.aside 
      initial={{ width: 280 }}
      animate={{ width: isOpen ? 280 : 0 }}
      className="h-screen bg-[#f0f4f9] border-r border-gray-200 overflow-hidden flex flex-col shrink-0"
    >
      <div className="p-4 flex items-center justify-between">
        <button className="flex items-center gap-2 text-sm font-medium text-gray-600 bg-gray-200/50 hover:bg-gray-200 px-3 py-2 rounded-lg transition-colors w-full">
          <Plus size={16} />
          <span>New Chat</span>
        </button>
      </div>

      <div className="flex-1 overflow-y-auto px-2 space-y-1">
        <div className="px-3 py-2 text-xs font-semibold text-gray-500">Recent</div>
        {[
            "React Performance Optimization", 
            "Weather in Tokyo", 
            "Debug Python Script",
            "Generate Marketing Copy"
        ].map((title, i) => (
          <button key={i} className="flex items-center gap-3 w-full px-3 py-2 text-sm text-gray-700 hover:bg-white rounded-lg transition-colors text-left group">
            <MessageSquare size={16} className="text-gray-400 group-hover:text-blue-500" />
            <span className="truncate">{title}</span>
          </button>
        ))}
      </div>

      <div className="p-4 border-t border-gray-200">
        <button className="flex items-center gap-3 w-full px-3 py-2 text-sm font-medium text-gray-700 hover:bg-white rounded-lg transition-colors">
            <div className="w-8 h-8 rounded-full bg-purple-600 flex items-center justify-center text-white font-bold">U</div>
            <div className="flex flex-col text-left">
                <span className="text-xs font-bold">Upgrade Plan</span>
                <span className="text-[10px] text-gray-500">Get Gemini Advanced</span>
            </div>
        </button>
      </div>
    </motion.aside>
  );
};

/**
 * 2. Tool Widget Component
 * Visualizes the "Thinking" or "Action" state of the AI.
 */
const ToolWidget = ({ tool }: { tool: ToolInvocation }) => {
