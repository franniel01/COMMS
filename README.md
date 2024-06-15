% Main Script for Synthesizer Application
function SynthesizerAppsss
    % Create the UI figure
    fig = uifigure('Position', [100 100 800 1000], 'Name', 'FQSK app');

    % UI Components
    % Dropdown for notes
    lblNote = uilabel(fig, 'Position', [50 2000 100 30], 'Text', 'Select Note:');
    ddNote = uidropdown(fig, 'Position', [150 700 100 30], 'Items', {'A','A#', 'B', 'C', 'D', 'E', 'F', 'G'});

    % Slider for duration
    lblDuration = uilabel(fig, 'Position', [50 2000 100 30], 'Text', 'Duration (s):');
    sldDuration = uislider(fig, 'Position', [150 660 100 3], 'Limits', [0.1 5], 'Value', 1);

    % Axes for plotting the waveform
    axSound = uiaxes(fig, 'Position', [50 450 700 150]);
    title(axSound, 'Sound Waveform');
    xlabel(axSound, 'Time (s)');
    ylabel(axSound, 'Amplitude');

    % Button to generate sound
    btnGenerate = uibutton(fig, 'Position', [50 600 100 30], 'Text', 'Generate Sound', ...
        'ButtonPushedFcn', @(btn,event) generateSound(ddNote.Value, sldDuration.Value, axSound));

    % FSK Modulation UI Components
    lblData = uilabel(fig, 'Position', [300 500 100 30], 'Text', 'Data Bits (0/1):');
    txtData = uitextarea(fig, 'Position', [400 700 100 30], 'Value', '101010');
    
    lblF1 = uilabel(fig, 'Position', [300 650 100 30], 'Text', 'Frequency 1:');
    txtF1 = uitextarea(fig, 'Position', [400 650 100 30], 'Value', '440');
    

    lblF2 = uilabel(fig, 'Position', [300 600 100 30], 'Text', 'Frequency 2:');
    txtF2 = uitextarea(fig, 'Position', [400 600 100 30], 'Value', '880');
    
    axFSK = uiaxes(fig, 'Position', [50 250 700 150]);
    title(axFSK, 'FSK Waveform');
    xlabel(axFSK, 'Time (s)');
    ylabel(axFSK, 'Amplitude');
    
    btnFSK = uibutton(fig, 'Position', [300 550 100 30], 'Text', 'FSK Modulate', ...
        'ButtonPushedFcn', @(btn,event) FSKModulateSound(txtData.Value, str2double(txtF1.Value), str2double(txtF2.Value), sldDuration.Value, axFSK));

    % QAM Modulation UI Components
    lblQAMData = uilabel(fig, 'Position', [300 400 100 30], 'Text', 'QAM Data Bits (0/1):');
    txtQAMData = uitextarea(fig, 'Position', [400 500 100 30], 'Value', '101010');
    
    lblQAMCarrier = uilabel(fig, 'Position', [300 450 100 30], 'Text', 'Carrier Frequency:');
    txtQAMCarrier = uitextarea(fig, 'Position', [400 450 100 30], 'Value', '1000');
    
    axQAM = uiaxes(fig, 'Position', [50 50 700 150]);
    title(axQAM, 'QAM Waveform');
    xlabel(axQAM, 'Time (s)');
    ylabel(axQAM, 'Amplitude');
    
    btnQAM = uibutton(fig, 'Position', [300 400 100 30], 'Text', 'QAM Modulate', ...
        'ButtonPushedFcn', @(btn,event) QAMModulateSound(txtQAMData.Value, str2double(txtQAMCarrier.Value), sldDuration.Value, axQAM));
end

% Function to generate sound for a given note and duration
function generateSound(note, duration, ax)
    fs = 44100; % Sample rate
    frequency = getNoteFrequency(note);
    y = generateHarmonics(frequency, duration, fs);
    plotWaveform(ax, y, fs);
    sound(y, fs);
end

% Function to get the frequency of a note
function frequency = getNoteFrequency(note)
    switch note
        case 'A'
            frequency = 440;
        case 'A#'
            frequency = 466.16;
        case 'B'
            frequency = 493.88;
        case 'C'
            frequency = 523.25;
        case 'D'
            frequency = 587.33;
        case 'E'
            frequency = 659.25;
        case 'F'
            frequency = 698.46;
        case 'G'
            frequency = 783.99;
        otherwise
            frequency = 440; % Default to A if unknown
    end
end

% Function to generate harmonics
function y = generateHarmonics(frequency, duration, fs)
    t = 0:1/fs:duration;
    y = sin(2*pi*frequency*t);
    for n = 2:8
        if n ~= 7 % Skipping the 7th harmonic
            y = y + (1/n) * sin(2*pi*n*frequency*t);
        end
    end
end

% Function for FSK modulation
function FSKModulateSound(dataStr, f1, f2, bit_duration, ax)
    fs = 44100; % Sample rate
    data = arrayfun(@(c) str2double(c), dataStr);
    y = FSKModulate(data, f1, f2, bit_duration, fs);
    plotWaveform(ax, y, fs);
    sound(y, fs);
end

% Function to perform FSK modulation
function y = FSKModulate(data, f1, f2, bit_duration, fs)
    t = 0:1/fs:bit_duration;
    y = [];
    for bit = data
        if bit == 0
            y = [y, sin(2*pi*f1*t)];
        else
            y = [y, sin(2*pi*f2*t)];
        end
    end
end

% Function for QAM modulation
function QAMModulateSound(dataStr, carrier_freq, bit_duration, ax)
    fs = 44100; % Sample rate
    data = arrayfun(@(c) str2double(c), dataStr);
    y = QAMModulate(data, carrier_freq, bit_duration, fs);
    plotWaveform(ax, y, fs);
    sound(y, fs);
end

% Function to perform QAM modulation
function y = QAMModulate(data, carrier_freq, bit_duration, fs)
    t = 0:1/fs:bit_duration;
    y = [];
    for bit = data
        if bit == 1
            yI = cos(2*pi*carrier_freq*t);
            yQ = sin(2*pi*carrier_freq*t);
        else
            yI = -cos(2*pi*carrier_freq*t);
            yQ = -sin(2*pi*carrier_freq*t);
        end
        y = [y, yI + yQ];
    end
end

% Function to plot the waveform
function plotWaveform(ax, y, fs)
    t = (0:length(y)-1) / fs;
    plot(ax, t, y);
    title(ax, 'Waveform');
    xlabel(ax, 'Time (s)');
    ylabel(ax, 'Amplitude');
end
